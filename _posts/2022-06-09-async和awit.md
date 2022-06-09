---
layout:     post   				    # 使用的布局（不需要改）
title:      async和await 				# 标题
subtitle:   关于async和await遇到的用法                  #副标题
date:       2022-06-09 				# 时间
author:     BY 	iron3000					# 作者
header-img: img/post-bg-2015.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - 学习
---
### 关于async和await
做毕设的时候有需求，promiseA请求的数据处理需要promiseB数据的支撑，那就需要等待promiseB返回后才处理promiseA。
```
//我写的是在promiseB的.then方法里面进行promiseA
getproduce().then(res=>{
	getDevice().then(){
    	this.is
    }

})
//实际使用async await最好
async function get(){
	await getproduce().then(res=>{
     this.is
    })
    getDevice().then(res=>{
    this.isis
    })
}

```
