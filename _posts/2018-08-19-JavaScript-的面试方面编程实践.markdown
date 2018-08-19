---
layout: post
title:  "JavaScript 的面向方面编程实践"
date:   2018-08-19 10:46:06 +0800
categories: 程序员日常
---

### 为什么要在 JavaScript 中应用面向方面的编程？
有一些代码逻辑，需要和业务逻辑分离出来，所以应用面向方面的编程比较自然。

比如`记录日志`、`发送通知拦截`等等。除了这两种场景，我还在实际的 `CRM` 系统中， 应用面向方面编程来控制用户的状态迁移。

`记录日志` 很容易理解，我们实际的使用场景是对于某些操作，会发送信息到 fundebug 平台。

`发送通知拦截`，是指在发送邮件、微信模板消息时，需要检查当前是否为正式环境，该用户是否允许接收通知等等情况。

`用户状态迁移`，也即系统中一个用户的整个生命周期。

以上这些场景，使用面向方面编程能够实现更好的关注点分离，第二个好处是，不需要对原有代码进行修改，而是添加新的文件，在引用原有文件后增加新的代码，并在入口文件里调用新文件即可。代码迭代的方式比较优雅。

### 如何应用？
在网上搜索之后，发现比较好的实现方式是通过在 `Function` 的 `prototype` 里增加通用的前置或者后置函数。然后，就是把原有方法使用这样的前置或者后置函数包装，这个前置或者后置函数接收一个函数参数，在这个函数参数中，可以添加自定义的代码，以实现在原来的函数执行前或者后，需要额外做的逻辑。

在实际应用中，需要针对 ES 6 以及 `async` 和 `await` 专门写一个版本。

### 代码示例
```javascript
// AOP.js
export default class AOP {
    static setBefore() {
        /* eslint-disable */
        Function.prototype.before = function (func) {
            const self = this
            return function () {
                if (func.apply(this, arguments) === false) {
                    return false;
                }

                return self.apply(this, arguments)
            }
        };

        Function.prototype.beforeAsync = function (asyncFunc) {
            const self = this
            return async function () {
                if (await asyncFunc.apply(this, arguments) === false) {
                    return false
                }

                return await self.apply(this, arguments)
            }
        }
        /* eslint-enable */
    }

    static setAfter() {
        /* eslint-disable */
        Function.prototype.afterAsync = function (asyncFunc) {
            const self = this;
            return async function () {
                let result = await self.apply(this, arguments)

                if (result === false) {
                    return false;
                }

                await asyncFunc.apply(self, [result, ...arguments]);
                return result;
            }
        }
        /* eslint-enable */
    }
}
```
应用：
```javascript
import AOP from './AOP'

// 原有代码：
function doSomething(){}

// 加 log
doSomething = doSomething.before(()=>{
    logger.info('will do something...')
})
```

```javascript
// 原有通知代码 wechat.js：
export default class wechat{
    static sendTemplateMessage(wechat_openid){}
}
```
```javascript
import wechat from './wechat.js'
// 通知拦截
wechat.sendTemplateMessage = wechat.sendTemplateMessage.beforeAsync(async (wechat_openid)=>{
    if(!inProduction || !(await getUserByOpenId(wechat_openid)).allowReceivingNotification){
        return false
    } 
    
    return true
})

// 把引用原来的 wechat 的地方，改成引用这个新的 wechat
module.exports = wechat
```

```javascript
// 原代码
async doSomething(userId){
    await xxx()
}

// 新代码
doSomething = doSomething.after(async (userId)=>{
    await tagUserAsState(userId, 'new state')
})
```

### 待做事项
将 `AOP.js` 封装成一个 `npm` 包，方便不同的项目使用。