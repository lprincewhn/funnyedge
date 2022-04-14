---
title: "网站加速"
date: 2022-04-12T14:32:23+08:00
weight: 50
draft: false
---

一个Web网站通常包括图片,html,css,js等静态文件，也包括用户登陆，数据查询，文件上传等服务接口调用，因此是静态内容和动态接口的混合场景。CloudFront在用户层面屏蔽了两种加速技术的差异，通过配置的方式实现全站的加速。本章在["动态加速"](/10_quickstart/40_dynamic/)的基础上，增加针对静态文件的缓存行为。

1. 参考["动态加速"](/10_quickstart/40_dynamic/)创建一个分配。

2. 点击分配ID进入详情页。
![修改分配](/images/modify_distribution.png?classes=border)

3. 切换到”缓存行为“标签页，点击”创建行为“按钮
![新增行为](/images/modify_distribution_behavior.png?classes=border)

    3.1 指定路径模式，设置源站和允许的HTTP方法。如：图片都存在images目录下，可使用路径模式“*/images/*”
    ![指定路径模式](/images/create_behavior.png?classes=border)
    3.2 选择缓存策略和源请求策略。
    ![指定路径模式](/images/create_behavior_policy.png?classes=border)
    3.3 点击"创建行为"完成。

4. 创建完成后，在“行为”标签页可以看到两个行为，这两个行为具有不同的路径模式，和用户请求的路径进行匹配是将按照优先顺序进行。
![指定路径模式](/images/view_behavior.png?classes=border)

5. 继续添加其他路径模式的行为，如*.html，*.js, *.css。

修改完成后，稍等几分钟待配置下发到全球所有边缘节点加速效果即可生效。配置生效后：

1. 假设用户访问地址：http://dxxxxx.cloudfront.net/abc/images/logo.jpg，由于其路径/abc/images/logo.jpg和第一个行为的路径模式\(\*/images/\*\)匹配，因此该请求结果将会被缓存。

2. 假设用户访问地址：http://dxxxxx.cloudfront.net/api/login，由于其路径/api/login和第一个行为的路径模式不匹配，因此继续匹配后续的行为，最后匹配到的是默认缓存行为，因此该请求结果不会缓存。





