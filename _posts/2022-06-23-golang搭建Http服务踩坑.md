---
layout:     post   				    # 使用的布局（不需要改）
title:      golang Http请求踩坑 		# 标题
subtitle:                          #副标题
date:       2022-06-23 				# 时间
author:     BY 	iron3000					# 作者
header-img: img/post-bg-2015.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - 学习
---
##golang 搭建Http服务相关
今天毕设设备认证服务启动后，设备端一直没有相应打印，然后也访问不到服务，很久找不到原因，解决后记录一下
```
http.HandleFunc("mqtt/auth", authFunc)
err := ListenAndServe("127.0.0.1:11888",nil)
if err != nil{
	fmt.Println(err)
}
fmt.Println("connect success")
```
上述代码无论如何都访问不到，也不报错，也不显示“connect success”，查询资料后发现，golang的ListenAndServe失败时，即`err != nil`时会往下执行，如果监听端口成功会阻塞当前代码，其下代码都不会执行了，所以不打印“connect success”。

而访问不到127.0.0.1:11888/mqtt/auth，原因是`http.HandleFunc("mqtt/auth", authFunc)`中路径参数需要以`/`开头，否则就访问不到。将代码修改为`http.HandleFunc("/mqtt/auth", authFunc)` 即可。