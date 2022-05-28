---
title: "使用Athena分析访问日志"
date: 2022-04-12T14:32:23+08:00
weight: 30
draft: false
---

### 1.（可选）创建用于存储CloudFront日志的存储桶

如果还没有用于存储日志的S3桶，按照以下步骤创建，如果要使用已有桶，可跳过该步骤。

  1.1 使用以下URL打开S3控制台
  
  https://s3.console.aws.amazon.com/s3/buckets
  ![创建S3桶](/images/s3_create_bucket.png?classes=border)

  1.2 输入存储桶名称和选择区域，其余参数保留默认配置
  ![S3桶参数](/images/s3_create_bucket_for_cloudfront.png?classes=border)

  1.3 为日志桶设置生命周期规则，定期删除旧文件。

  使用Athena查询时费用和扫描的数据量有关，因此定期删除过期文件，可控制查询成本以及提高查询速度。

  在S3桶的详情页面切换到“管理”标签页，点击“创建生命周期规则”
  ![S3桶管理](/images/s3_bucket_manage.png?classes=border)
  ![S3桶生命周期规则](/images/s3_create_lifecycle_rule.png?classes=border)

  参考以下截图进行配置:
  ![S3桶生命周期规则详情](/images/s3_lifecycle_detail1.png?classes=border)
  ![S3桶生命周期管理详情](/images/s3_lifecycle_detail2.png?classes=border)

### 2. 打开CloudFront分配标准日志

  2.1 点击分配ID进入详情页，确认标准日志已经打开
  ![分配详情](/images/cloudfront_standard_log.png?classes=border)

  2.2 点击“编辑”按钮进入编辑页面，启用标准日志，确认或设置日志存储路径。
  ![打开标准日志](/images/cloudfront_log_destication.png?classes=border)

### 3. 通过Athena使用SQL分析日志

通过以下链接打开Athena控制台, 切换到和日志存储桶相同的区域
  
https://console.aws.amazon.com/athena/home
  
  3.1 (可选) 如果是第一次使用Athena，需要先设置一个S3桶用于存储Athena查询的结果，步骤如下图：
  ![查看Athena设置](/images/athena_setting.png?classes=border)
  ![修改Athena设置](/images/athena_setting_manage.png?classes=border)
  ![设置Athena临时存储桶](/images/athena_setting_bucket.png?classes=border)

  3.2 在Athena查询编辑器
  ![Athena查询编辑器](/images/athena_query.png?classes=border)

  3.3 运行建表语句

  在查询编辑器中运行以下建表语句，注意需要将其中的LOCATION修改为存储CloudFront标准日志的路径。
  
    CREATE EXTERNAL TABLE IF NOT EXISTS default.cloudfront_logs (
      `date` DATE,
      time STRING,
      location STRING,
      bytes BIGINT,
      request_ip STRING,
      method STRING,
      host STRING,
      uri STRING,
      status INT,
      referrer STRING,
      user_agent STRING,
      query_string STRING,
      cookie STRING,
      result_type STRING,
      request_id STRING,
      host_header STRING,
      request_protocol STRING,
      request_bytes BIGINT,
      time_taken FLOAT,
      xforwarded_for STRING,
      ssl_protocol STRING,
      ssl_cipher STRING,
      response_result_type STRING,
      http_version STRING,
      fle_status STRING,
      fle_encrypted_fields INT,
      c_port INT,
      time_to_first_byte FLOAT,
      x_edge_detailed_result_type STRING,
      sc_content_type STRING,
      sc_content_len BIGINT,
      sc_range_start BIGINT,
      sc_range_end BIGINT
    )
    ROW FORMAT DELIMITED 
    FIELDS TERMINATED BY '\t'
    LOCATION 's3://<CloudFront_standard_log_bucket_name/prefix/>'
    TBLPROPERTIES ( 'skip.header.line.count'='2' )

  ![Athena查询编辑器](/images/athena_create_table.png?classes=border)

  3.4 运行SQL查询
  ![Athena查询CloudFront标准日志](/images/athena_query_cloudfront_log.png?classes=border)

  CloudFront标准日志各个字段的说明参考以下链接：

  https://docs.aws.amazon.com/zh_cn/AmazonCloudFront/latest/DeveloperGuide/AccessLogs.html#BasicDistributionFileFormat
  



