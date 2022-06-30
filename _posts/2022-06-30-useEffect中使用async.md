---
layout:     post   				    # 使用的布局（不需要改）
title:      useEffect中使用async函数 		# 标题
subtitle:                          #副标题
date:       2022-06-30 				# 时间
author:     BY 	iron3000					# 作者
header-img: img/post-bg-2015.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - 学习
---
# 在useEffect中使用async函数
useEffect经常用来在react中获取远端数据。获取数据一般是一个promise异步函数，在useEffect中使用他们可能不会像我们想象中的那么直接。

#### 错误的方式
```
// ❌ don't do this
useEffect(async () => {
  const data = await fetchData();
}, [fetchData])
```
这样错误的原因是，useEffect这个第一个参数需要传入的函数是一个返回值为undefined或者一个处理函数（用来清除副作用）的函数（有点绕口），但是async函数返回的是一个promise，意味着他不能像函数一样被调用，这就和useEffect所期望的第一个参数不同。

#### 直接将获取data的代码写在useEffect回调参数的函数体里面
一般最直接的方式就是使用这种
```
useEffect(() => {
  // declare the data fetching function
  const fetchData = async () => {
    const data = await fetch('https://yourapi.com');
  }

  // call the function
  fetchData()
    // make sure to catch any error
    .catch(console.error);
}, [])
```

#### 如果获取data的代码必须写在useEffect外面那该怎么办呢
如果你必须把你的数据获取函数写在useEffect外面，那你就需要使用useCallback来包裹你的数据获取函数。（当然不绝对，我项目中遇到过包裹出错不包裹才能使用呢的情况，这里的意思是如果你useEffect依赖项是这个获取data的函数）
为什么呢？如果是普通函数，他将会在每次re-render中更新，而你的useEffect依赖这个函数，这样就会出现你不期望出现的情况。其实就是如果你不期望你的函数在每次更新都重新创建的话，就用useCallback包裹他
```
// declare the async data fetching function
const fetchData = useCallback(async () => {
  const data = await fetch('https://yourapi.com');

  setData(data);
}, [])

// the useEffect is only there to call `fetchData` at the right time
useEffect(() => {
  fetchData()
    // make sure to catch any error
    .catch(console.error);;
}, [fetchData])

```
#### 获取data都函数在useEffect中时需要注意的事
之前获取data函数在useEffect中的例子，有许多值得注意点事，这个如果在你的useEffect没有依赖项的时候没有影响，但是如果有依赖，当依赖改变的时候你的useEffect会调用两次，而当这两次发生的非常快时，第一次和第二次请求就会发起竞争，如果第一次跑赢了，你在里面set状态的时候就会将过时的值set进去。
```
useEffect(() => {
  // declare the async data fetching function
  const fetchData = async () => {
    // get the data from the api
    const data = await fetch(`https://yourapi.com?param=${param}`);
    // convert the data to json
    const json = await response.json();

    // set state with the result
    setData(json);
  }

  // call the function
  fetchData()
    // make sure to catch any error
    .catch(console.error);;
}, [param])
```

可以这样解决
```
useEffect(() => {
  let isSubscribed = true;

  // declare the async data fetching function
  const fetchData = async () => {
    // get the data from the api
    const data = await fetch(`https://yourapi.com?param=${param}`);
    // convert the data to json
    const json = await response.json();

    // set state with the result if `isSubscribed` is true
    if (isSubscribed) {
      setData(json);
    }
  }

  // call the function
  fetchData()
    // make sure to catch any error
    .catch(console.error);;

  // cancel any future `setData`
  return () => isSubscribed = false;
}, [param])
```