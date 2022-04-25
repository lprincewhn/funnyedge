---
title: "利用标签进行权限限制和费用分摊"
date: 2022-04-12T14:32:23+08:00
weight: 10
draft: false
---

### 0. 打开CloudShell

https://console.aws.amazon.com/cloudshell/home

CloudShell在浏览器上提供了一个运行各种aws命令行工具的的shell环境，使得你可以非常简单的使用命令管理和操作AWS资源。CloudShell直接继承登陆控制台用户的权限，无需额外使用AKSK，从而避免了AKSK泄露的风险。

![CloudShell](/images/cloudshell.png?classes=border)

**小技巧：在控制台右上角的“操作”菜单,通过上传和下载文件可进行本地和CloudShell之间的文件传输。**

    # 下载并解压AutoOps软件包
    cd ~
    wget https://github.com/lprincewhn/AutoOps/archive/refs/tags/v0.9.0-alpha.220423.zip
    unzip v0.9.0-alpha.220423.zip


### 1. 创建CloudTrail跟踪

通过以下命令部署AutoOps通用运维组件，包括：

- 一个SNS主题，用于发送事件通知
- 一个S3桶，用于存放CloudTrail跟踪文件
- 一个CloudTrail跟踪，监控本账号所有区域的管理事件，创建跟踪后，可以通过EventBridge监控API调用事件

```
cd ~/AutoOps-0.9.0-alpha.220423/Common  
MAIN_REGION=us-east-1
sam build
sam deploy --stack-name AutoOpsCommon --region $MAIN_REGION --confirm-changeset   --resolve-s3 --capabilities CAPABILITY_IAM
``` 

部署过程中，屏幕将会提示本次部署所需要创建/更新/删除的组件，确认后输入“y“->“回车”继续完成部署。

### 2. 部署CloudFront自动配置流程

该流程将通过EventBridge侦听CloudFront分配的创建，修改和删除事件，对分配的标签和告警进行配置。流程图如下：

![CloudFrontProvision](/images/CloudFrontProvision.png?classes=border)

标签包括：
- 将分配的“备用域名”拷贝到分配的DomainName标签中
- 将操作用户的用户名或角色名称拷贝到分配的Owner标签中
- 将操作用户的特定标签（默认为Project和Team）拷贝到分配的相应标签中

告警包括：
- 5xx错误率>5%
- 请求次数突降>80%

命令如下：

```
cd ~/AutoOps-0.9.0-alpha.220423/CloudFrontProvision
REGION=us-east-1
cp workflows/state.asl.all.json state.asl.json 
sam build
sam deploy --stack-name AutoOpsCloudFrontProvision --region $REGION --confirm-changeset --resolve-s3 --capabilities CAPABILITY_IAM
```

部署过程中，屏幕将会提示本次部署所需要创建/更新/删除的组件，确认后输入“y“->“回车”继续完成部署。

### 3. 使用标签分析成本

3.1 激活成本标签

点击右上角的登陆用户信息弹出下拉框，选择“账单控制面板“看看账单和费用信息

![打开账单控制面板](/images/billing_dashboard.png?classes=border&width=200px)

在左侧导航栏选择“成本分配标签”

![激活成本标签](/images/active_cost_tag.png?classes=border&width=700px)

3.2 在Cost Explorer中查看账单

使用以下URL打开Cost Explorer控制台：
    
https://console.aws.amazon.com/cost-management/home#/custom

在图表右侧的筛选条件中，可使用标签以及其他维度对关心的费用进行过滤：

![Cost Explorer筛选条件](/images/ce_filter.png?classes=border&width=200px)

在图表上方的分组依据中，也可使用标签以及其他维度进行费用的汇总：

![Cost Explorer分组条件](/images/ce_groupby.png?classes=border&width=600px)

### 4. 使用标签限制权限

4.1 创建策略
    
    # 定义策略文档：当资源上面的标签（aws:ResourceTag/Project）存在，且不等于操作用户的标签（aws:PrincipalTag/Project）时拒绝请求
    cat <<EoF > cloudfront-deny-by-project-policy.json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Deny",
                "Action": "cloudfront:*",
                "Resource": "*",
                "Condition": {
                        "StringNotEquals": {
                        "aws:ResourceTag/Project": "${aws:PrincipalTag/Project}"
                    },
                    "Null": {
                        "aws:ResourceTag/Project": "false"
                    }
                }
            }
        ]
    }
    EoF

    # 创建策略并将其ARN保存在环境变量
    POLICY_ARN=$(aws iam create-policy --policy-name CloudFrontDenyByProjectPolicy --policy-document file://cloudfront-deny-by-project-policy.json --query Policy.Arn --output text)

    echo $POLICY_ARN

4.2 给test用户附加策略

    aws iam attach-user-policy --user-name test --policy-arn ${POLICY_ARN}

4.3 给test用户分配标签

    aws iam tag-user --user-name test --tags Key=Project,Value=abcd

4.4 使用test用户测试

![打开账单控制台](/images/cloufront_deny.png?classes=border)

可以看到，虽然test用户仍然可以列出所有分配，但是针对指定分配无法查看详情或者修改。



