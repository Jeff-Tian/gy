---
title: 欺骗 JavaScript 的 await
layout: post
date: '2018-10-08 10:50:51 +0800'
categories: 程序员日常 
---

### 背景
原来 `JavaScript` 的 `await` 关键字，不仅可以用于 `Promise`，`async` 函数，还可以用于任何一个带有 `then` 方法的对象。

### 实验
[打开这个页面在线运行实验](http://jsbin.com/yufefij/edit?html,console,output)

比如下面这段代码，输入结果是这样的：
```bash
started
then
done with 100
async over
```

源代码如下：
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width">
  <title>JS Bin</title>
</head>
<body>
  <button onclick="start()">Start</button>
  <script type="text/javascript">
    async function start() {
      console.log('started');

      try {
        const res = await fakeAsyncCompute;
        console.log('done with ', res);
      } catch (ex) {
        console.log('ex = ', ex);
      } finally {
        console.log('async over');
      }
    }
        
    const fakeAsyncCompute = {
      then: (successCallback, errorCallback)=>{
        setTimeout(()=> {
          console.log('then'); 
          // 通过调用 successCallback 可以将 try 块里的代码执行完
          successCallback(100);
          // 如果想将执行流程跳转到上面的 catch 块，可以调用 errorCallback
          // errorCallback(200);
        }, 3000);
      }
    }
  </script>
</body>
</html>
```