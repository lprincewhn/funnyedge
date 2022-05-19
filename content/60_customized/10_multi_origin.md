---
title: "根据客户端所在国家选择源站"
date: 2022-04-12T14:22:31+08:00
weight: 10
draft: false
---

Lambda@Edge是Amazon CloudFront的一个功能，它可让您在靠近应用程序用户的地方运行代码，从而提高性能，降低延迟。使用 Lambda@Edge，您无需在全球多个地方预置或管理基础设施。您只需按使用的计算时间付费——代码未运行时不产生费用。

使用Lambda@Edge，您可以将Web应用程序分布在全球并提高它们的性能（并且无需管理任何服务器），从而丰富您的Web应用程序。Lambda@Edge根据Amazon CloudFront内容分发网络(CDN)生成的事件运行代码。您只需将代码上传到AWS Lambda，后者将在靠近最终用户的AWS站点完成运行和扩展代码所需的一切操作，从而实现高可用性。

Lambda@Edge允许您运行Node.js和Python函数来自定义CloudFront提供的内容，从而在更靠近查看器的AWS位置执行函数。这些函数在不调配或管理服务器的情况下运行，以响应CloudFront事件。您可以在以下时间点使用Lambda函数来更改CloudFront请求和响应：

- 在CloudFront收到查看器的请求时（查看器请求）
- 在CloudFront将请求转发到源之前（源请求）
- 在CloudFront收到来自源的响应之后（源响应）
- 在CloudFront将响应转发到查看器之前（查看器响应）

![修改分配](/images/cloudfront-events-that-trigger-lambda-functions.png?classes=border)

1. 创建Lambda@Edge函数

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

1.2 创建lambda及版本

    # 创建函数
    zip -r multi-origin.zip lambda_function.py
    FUNCTION_NAME=edge-multi-origin
    aws lambda create-function --function-name $FUNCTION_NAME --runtime 'python3.7' --role $ROLE_ARN --handler lambda_function.lambda_handler --zip-file fileb://multi-origin.zip --no-cli-pager --region us-east-1

    # 发布版本 
    LAMBDA_EDGE_ARN=$(aws lambda publish-version --function-name $FUNCTION_NAME --no-cli-pager --output text --query 'FunctionArn' --region us-east-1)
    echo $LAMBDA_EDGE_ARN

2. 关联CloudFront行为和Lambda@Edge函数

    2.1 点击CloudFront分配ID进入详情页。
    ![修改分配](/images/modify_distribution.png?classes=border)

    2.2 切换到”缓存行为“标签页，选择需要修改的行为，点击”编辑“按钮
    ![修改行为](/images/modify_behaviour.png?classes=border)

    2.3 在行为详情页的底部“函数关联“，将之前创建的Lambda@Edge关联到源请求事件上
    ![函数关联](/images/assocaite_lambda_edge.png?classes=border)

3. 设置回源标头，作为Lambda@Edge函数的入参

    2.1 点击CloudFront分配ID进入详情页。
    ![修改分配](/images/modify_distribution.png?classes=border)

    2.2 切换到”源“标签页，选择需要修改的源，点击”编辑“按钮
    ![修改源](/images/modify_origin.png?classes=border)

    2.3 在“添加自定义标头“部分，根据需求添加以下标头
    ![添加自定义标头](/images/add_origin_header.png?classes=border)

    这些标头将作为Lambda@Edge函数的入参控制其行为，参数说明如下：
    - lambda-debug，控制日志输出，只要存在该标头，则函数输出更详细的Debug日志
    - countries-list, 国家组，以国家ISO二位编码表示，相同源站的国家为一组，组之间以逗号分隔，组内国家之间以竖线分隔
    - origin-list，源站列表，数量需要和国家组一样，位置上一一对应
