---
title: "根据客户端所在国家选择源站"
date: 2022-04-12T14:22:31+08:00
weight: 10
draft: false
---

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
    curl -o multi-origin.zip http://localhost:1313/multi-origin.zip
    FUNCTION_NAME=edge-multi-origin
    aws lambda create-function --function-name $FUNCTION_NAME --runtime 'python3.7' --role $ROLE_ARN --handler lambda_function.lambda_handler --zip-file fileb://multi-origin.zip --no-cli-pager --region us-east-1

    # 发布版本 
    LAMBDA_EDGE_ARN=$(aws lambda publish-version --function-name $FUNCTION_NAME --no-cli-pager --output text --query 'FunctionArn' --region us-east-1)
    echo $LAMBDA_EDGE_ARN

### 2. 关联Lambda@Edge函数

2.1 点击CloudFront分配ID进入详情页。
![修改分配](/images/modify_distribution.png?classes=border)

2.2 切换到”缓存行为“标签页，选择需要修改的行为，点击”编辑“按钮
![修改行为](/images/modify_behaviour.png?classes=border)

2.3 在行为详情页的底部“函数关联“，将之前创建的Lambda@Edge关联到源请求事件上
![函数关联](/images/assocaite_lambda_edge.png?classes=border)

### 3. 设置回源标头

3.1 点击CloudFront分配ID进入详情页。
![修改分配](/images/modify_distribution.png?classes=border)

3.2 切换到”源“标签页，选择需要修改的源，点击”编辑“按钮
![修改源](/images/modify_origin.png?classes=border)

3.3 在“添加自定义标头“部分，根据需求添加以下标头
![添加自定义标头](/images/add_origin_header.png?classes=border)

这些标头将作为Lambda@Edge函数的入参控制其行为，参数说明如下：
- lambda-debug，控制日志输出，只要存在该标头，则函数输出更详细的Debug日志
- countries-list, 国家组，以国家ISO二位编码表示，相同源站的国家为一组，组之间以逗号分隔，组内国家之间以竖线分隔
- origin-list，源站列表，数量需要和国家组一样，位置上一一对应
