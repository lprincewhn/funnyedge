---
title: "根据客户端所在国家选择源站"
date: 2022-04-12T14:22:31+08:00
weight: 10
draft: false
---

### 0. 打开CloudShell

https://console.aws.amazon.com/cloudshell/home

CloudShell在浏览器上提供了一个运行各种aws命令行工具的的shell环境，使得你可以非常简单的使用命令管理和操作AWS资源。CloudShell直接继承登陆控制台用户的权限，无需额外使用AKSK，从而避免了AKSK泄露的风险。

![CloudShell](/images/cloudshell.png?classes=border)

**小技巧：在控制台右上角的“操作”菜单,通过上传和下载文件可进行本地和CloudShell之间的文件传输。**

### 1. 创建Lambda@Edge函数

1.1 创建策略
    
    # 定义角色信任策略文档：该策略将用于创建一个角色，并且允许Lambda及Lambda@Edge切换至该角色
    cat <<EoF > lambdaedge-assume-role-policy.json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Principal": {
                    "Service": [
                        "edgelambda.amazonaws.com",
                        "lambda.amazonaws.com"
                    ]
                },
                "Action": "sts:AssumeRole"
            }
        ]
    }
    EoF

    # 创建角色并将其ARN保存在环境变量
    ROLE_NAME=lambda-edge-multi-origin-role
    ROLE_ARN=$(aws iam create-role --role-name $ROLE_NAME --assume-role-policy-document file://lambdaedge-assume-role-policy.json --query Role.Arn --no-cli-pager --output text)
    echo $ROLE_ARN

    # 定义权限策略文档：该策略将允许上述角色往所有区域的cloudwatch写入日志
    cat <<EoF > lambdaedge-write-cloudwatch-log-policy.json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "logs:CreateLogGroup",
                    "logs:CreateLogStream",
                    "logs:PutLogEvents"
                ],
                "Resource": [
                    "arn:aws:logs:*:*:*"
                ]
            }
        ]
    }
    EoF

    # 创建策略并将其ARN保存在环境变量
    POLICY_ARN=$(aws iam create-policy --policy-name LambdaEdgeWriteCloudWatchLogPolicy --policy-document  file://lambdaedge-write-cloudwatch-log-policy.json --query Policy.Arn --no-cli-pager --output text)
    echo $POLICY_ARN

    # 附加所需权限
    aws iam attach-role-policy --role-name $ROLE_NAME --policy-arn $POLICY_ARN

1.2 创建Lambda函数及版本

函数：[multi-origin.zip](/multi-origin.zip)

    # 创建函数
    wget https://funnyedge.svhw.tech/multi-origin.zip
    FUNCTION_NAME=edge-multi-origin
    aws lambda create-function --function-name $FUNCTION_NAME --runtime 'python3.7' --role $ROLE_ARN --handler lambda_function.lambda_handler --zip-file fileb://multi-origin.zip --no-cli-pager --region us-east-1

    # 发布版本 
    LAMBDA_EDGE_ARN=$(aws lambda publish-version --function-name $FUNCTION_NAME --no-cli-pager --output text --query 'FunctionArn' --region us-east-1)
    echo $LAMBDA_EDGE_ARN

### 2. 修改缓存行为

2.1 创建源请求策略，该策略将在回源时添加CloudFront-Viewer-Country标头

    # 定义源请求策略：以下示例基于动态加速的源请求策略修改，在转发所有标头，Cookie，查询字符串的基础上加上CloudFront-Viewer-Country以指示客户端所在国家
    cat <<EoF > all-viewer-plus-cloudfront-viewer-country.json
    {
        "Comment": "Add CloudFront header CloudFront-Viewer-Country in origin request.",
        "Name": "all-viewer-plus-cloudfront-viewer-country",
        "HeadersConfig": {
            "HeaderBehavior": "allViewerAndWhitelistCloudFront",
            "Headers": {
            "Quantity": 1,
            "Items": ["CloudFront-Viewer-Country"]
            }
        },
        "CookiesConfig": {
            "CookieBehavior": "all"
        },
        "QueryStringsConfig": {
            "QueryStringBehavior": "all"
        }
    }
    EoF

    # 创建源请求策略
    aws cloudfront create-origin-request-policy --origin-request-policy-config file://all-viewer-plus-cloudfront-viewer-country.json --no-cli-pager

2.2 点击CloudFront分配ID进入详情页。
![修改分配](/images/modify_distribution.png?classes=border)

2.3 切换到”缓存行为“标签页，选择需要修改的行为，点击”编辑“按钮
![修改行为](/images/modify_behaviour.png?classes=border)

2.4 在“缓存键和源请求”中设置源请求策略
![设置源请求策略](/images/change-origin-request-policy.png?classes=border&width=500)

2.5 在“函数关联“中，关联之前创建的Lambda@Edge关联到源请求事件上
![函数关联](/images/assocaite_lambda_edge.png?classes=border&width=500)

### 3. 设置回源标头

使用HTTP标头作为Lambda@Edge函数的入参控制其行为，参数说明如下：

- lambda-debug，控制日志输出，只要存在该标头，则函数输出更详细的Debug日志
- countries-list, 国家组，以国家ISO二位编码表示，相同源站的国家为一组，组之间以逗号分隔，组内国家之间以竖线分隔
- origin-list，源站列表，数量需要和国家组一样，位置上一一对应

3.1 点击CloudFront分配ID进入详情页。
![修改分配](/images/modify_distribution.png?classes=border)

3.2 切换到”源“标签页，选择需要修改的源，点击”编辑“按钮
![修改源](/images/modify_origin.png?classes=border)

3.3 在“添加自定义标头“部分，根据需求添加以下标头
![添加自定义标头](/images/add_origin_header.png?classes=border&width=500)


### 4. 查看日志

**注意：虽然所有Lambda@Edge函数均部署在us-east-1区域，但其运行在各个区域边缘，因此日志将记录在各个区域的CloudWatch中，也是这个原因，所以Lambda@Edge使用的角色权限策略需要涵盖各个区域的CloudWatch日志写权限**

![Lambda@Edge日志](/images/edge-multi-origin-log.png?classes=border)

