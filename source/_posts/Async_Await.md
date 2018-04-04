title: 使用Async/Await优化Promise
date: 2017/10/17
categories:
- javscript
---
（本来想吐槽下去年才熟练使用Promise但是又要用async-await，但是async-await炒鸡好用，默默流泪）
### Async/Await与Promise的关系
  async-await是promise和generator的语法糖，意在优化promise的写法，简单的说async-await是用来简化pormise写法的，并不是取代关系。

### Async/Await基本语法
  Async放在function关键字前面，await放在Async关键字的函数里，并且最终返回一个promise对象。如下代码所示：
  ```
   async function asyncAwaitDemo(){
       const result = await new Promise((resolve, reject)=>{
           setTimeout(()=>{
               resolve('test')
           },200)
       })
       return result;
   }
   asyncAwaitDemo().then((r)=>{
       console.log('result:',r);
   })
   // result: test
  ```
<!--more-->


  需要注意的是await只能放在async关键声明的函数里（这点和yield与*的关系一致）
  ```
    // 正常 for 循环
    async function demo() {
        let arr = [1, 2, 3, 4, 5];
        for (let i = 0, len = arr.length; i < len; i ++) {
            await arr[i];
        }
    }
    demo();//正常输出
    //如果for循环写成下面这样
    async function bugDemo() {
        let arr = [1, 2, 3, 4, 5];
        arr.forEach(item => {
            await item;
        });
    }
    bugDemo();// Uncaught SyntaxError: Unexpected identifier

  ```

  ### 场景示例
  这里写一个简单模拟异步api
  ```
  class Api {
  constructor () {
    this.user = { id: 1, name: 'test' }
    this.friends = [ this.user, this.user, this.user ]
    this.photo = 'not a real photo'
  }
  getUser () {
    return new Promise((resolve, reject) => {
      setTimeout(() => resolve(this.user), 200)
    })
  }
  getFriends (userId) {
    return new Promise((resolve, reject) => {
      setTimeout(() => resolve(this.friends.slice()), 200)
    })
  }
  getPhoto (userId) {
    return new Promise((resolve, reject) => {
      setTimeout(() => resolve(this.photo), 200)
    })
  }
  throwError () {
    return new Promise((resolve, reject) => {
      setTimeout(() => reject(new Error('Intentional Error')), 200)
    })
  }
}
  ```
  #### 需求1：获得用户信息
  promise链写法
  ```
  function promiseChain () {
  const api = new Api()
  let user, friends
  api.getUser()
    .then((returnedUser) => {
      user = returnedUser
      return api.getFriends(user.id)
    })
    .then((returnedFriends) => {
      friends = returnedFriends
      return api.getPhoto(user.id)
    })
    .then((photo) => {
      console.log('promiseChain', { user, friends, photo })
    })
}
  ```

Async/Await写法
```
async function asyncAwaitIsYourNewBestFriend () {
  const api = new Api()
  const user = await api.getUser()
  const friends = await api.getFriends(user.id)
  const photo = await api.getPhoto(user.id)
  console.log('asyncAwaitIsYourNewBestFriend', { user, friends, photo })
}
```

#### 需求2：获得朋友的朋友
 promise链写法:这里使用的是迭代循环
 ```
 function promiseLoops () {  
  const api = new Api()
  api.getUser()
    .then((user) => {
      return api.getFriends(user.id)
    })
    .then((returnedFriends) => {
      const getFriendsOfFriends = (friends) => {
        if (friends.length > 0) {
          let friend = friends.pop()
          return api.getFriends(friend.id)
            .then((moreFriends) => {
              console.log('promiseLoops', moreFriends)
              return getFriendsOfFriends(friends)
            })
        }
      }
      return getFriendsOfFriends(returnedFriends)
    })
}
 ```
 Async/Await写法:
 ```
 async function asyncAwaitLoops () {
  const api = new Api()
  const user = await api.getUser()
  const friends = await api.getFriends(user.id)
  for (let friend of friends) {
    let moreFriends = await api.getFriends(friend.id)
    console.log('asyncAwaitLoops', moreFriends)
  }
}
 ```
  Async/Await写法简化并发
  ```
  async function asyncAwaitLoopsParallel () {
  const api = new Api()
  const user = await api.getUser()
  const friends = await api.getFriends(user.id)
  const friendPromises = friends.map(friend => api.getFriends(friend.id))
  const moreFriends = await Promise.all(friendPromises)
  console.log('asyncAwaitLoopsParallel', moreFriends)
}
  ```
  参考：
  * [Promises/A+](https://promisesaplus.com/)
  * [ASYNC/AWAIT WILL MAKE YOUR CODE SIMPLER](https://blog.patricktriest.com/what-is-async-await-why-should-you-care/)
