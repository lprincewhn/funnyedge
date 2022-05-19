---
title: "在CloudWatch查看指标"
date: 2022-04-12T14:32:23+08:00
weight: 10
draft: false
---

CloudWatch是AWS的统一可观测性平台，可在此处对所有AWS服务和资源进行监控。

CloudFront的指标会被送到“弗吉尼亚北部”的CloudWatch当中，因此通过以下URL打开CloudWatch控制台：
https://console.aws.amazon.com/cloudwatch/home?region=us-east-1#

### 创建CloudWatch控制面板

![CloudWatch控制面板](/images/cloudwatch_dashboard.png?classes=border)

AWS已经提供了一些预定义的控制面板，这些控制面板在”自动控制面板“中，用户可以根据自己需求创建”自定义控制面板“，切换至”自定义控制面板“标签页，点击”创建控制面板“，在弹出的对话框中输入控制面板名称创建控制面板。

![自定义控制面板](/images/customerized_dashboard.png?classes=border)

![新建控制面板](/images/new_dashboard.png?classes=border)

### 在控制面板中添加指标面板

点击“添加小组件“按钮，选择”线形图“和“指标”

![添加小组件](/images/add_widget.png?classes=border)

![线形图](/images/line_chart.png?classes=border)

![指标和日志](/images/metric_line_chart.png?classes=border)










