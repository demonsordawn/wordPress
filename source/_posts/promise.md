---
title: "Promise 有几种状态？手写一个符合 Promise A+ 规范的 Promise"
date: 2025-10-29
tag: ['promise']
---
### Promise 有几种状态？
1. Pending: 初始状态，表示 Promise 实例的初始状态，既没有被执行也没有被拒绝。
2. Fulfilled: 成功状态，表示 Promise 实例的执行成功，对应于 resolve() 方法的调用。
3. Rejected: 失败状态，表示 Promise 实例的执行失败，对应于 reject() 方法的调用。

### 手写一个符合 Promise A+ 规范的 Promise
```javascript
class MyPromise {
  // 定义三种状态常量
  static PENDING = 'pending'
  static FULFILLED = 'fulfilled'
  static REJECTED = 'rejected'

  constructor(executor) {
    this.status = MyPromise.PENDING // 初始状态为pending
    this.value = undefined // 成功状态的值
    this.reason = undefined // 失败状态的原因          
    this.onFulfilledCallbacks = [] // 成功回调队列
    this.onRejectedCallbacks = [] // 失败回调队列

    // 成功处理函数
    const resolve = (value) => {
      // 只有pending状态可以转换为fulfilled
      if (this.status === MyPromise.PENDING) {
        this.status = MyPromise.FULFILLED
        this.value = value
        // 执行所有成功回调
        this.onFulfilledCallbacks.forEach(callback => callback())
      }
    }

    // 失败处理函数
    const reject = (reason) => {
      // 只有pending状态可以转换为rejected
      if (this.status === MyPromise.PENDING) {
        this.status = MyPromise.REJECTED
        this.reason = reason
        // 执行所有失败回调
        this.onRejectedCallbacks.forEach(callback => callback())
      }
    }

    try {
      // 立即执行 executor
      executor(resolve, reject)
    } catch (error) {
      // 执行器出错直接reject
      reject(error)
    }
  }

  // then方法实现
  then(onFulfilled, onRejected) {
    // 规范要求：如果onFulfilled不是函数，忽略它并透传value
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value
    // 规范要求：如果onRejected不是函数，忽略它并透传reason
    onRejected = typeof onRejected === 'function' ? onRejected : reason => { throw reason }

    // 返回新的Promise实现链式调用
    const promise2 = new MyPromise((resolve, reject) => {
      // 处理fulfilled状态
      if (this.status === MyPromise.FULFILLED) {
        // 规范要求：onFulfilled/onRejected必须异步执行（使用微任务模拟）
        queueMicrotask(() => {
          try {
            const x = onFulfilled(this.value)
            // 处理返回值x，实现不同类型的转换
            MyPromise.resolvePromise(promise2, x, resolve, reject)
          } catch (error) {
            reject(error)
          }
        })
      }

      // 处理rejected状态
      if (this.status === MyPromise.REJECTED) {
        queueMicrotask(() => {
          try {
            const x = onRejected(this.reason)
            MyPromise.resolvePromise(promise2, x, resolve, reject)
          } catch (error) {
            reject(error)
          }
        })
      }

      // 处理pending状态（收集回调）
      if (this.status === MyPromise.PENDING) {
        this.onFulfilledCallbacks.push(() => {
          queueMicrotask(() => {
            try {
              const x = onFulfilled(this.value)
              MyPromise.resolvePromise(promise2, x, resolve, reject)
            } catch (error) {
              reject(error)
            }
          })
        })

        this.onRejectedCallbacks.push(() => {
          queueMicrotask(() => {
            try {
              const x = onRejected(this.reason)
              MyPromise.resolvePromise(promise2, x, resolve, reject)
            } catch (error) {
              reject(error)
            }
          })
        })
      }
    })

    return promise2
  }

  // 处理Promise返回值的工具方法（核心规范）
  static resolvePromise(promise2, x, resolve, reject) {
    // 如果x和promise2是同一个对象，直接reject
    if (x === promise2) {
      return reject(new TypeError('Chaining cycle detected for promise'))
    }

    // 如果x是Promise实例
    if (x instanceof MyPromise) {
      x.then(
        value => MyPromise.resolvePromise(promise2, value, resolve, reject),
        reason => reject(reason)
      )
    } 
    // 如果x是对象或函数
    else if (x !== null && (typeof x === 'object' || typeof x === 'function')) {
      let then
      try {
        then = x.then // 取x的then方法
      } catch (error) {
        return reject(error)
      }

      // 如果then是函数，视为thenable对象
      if (typeof then === 'function') {
        let called = false // 防止多次调用
        try {
          then.call(
            x,
            y => {
              if (called) return
              called = true
              MyPromise.resolvePromise(promise2, y, resolve, reject) // 递归处理
            },
            r => {
              if (called) return
              called = true
              reject(r)
            }
          )
        } catch (error) {
          if (called) return
          called = true
          reject(error)
        }
      } else {
        // then不是函数，直接resolve x
        resolve(x)
      }
    } else {
      // x是普通值，直接resolve
      resolve(x)
    }
  }

  // catch方法（语法糖，相当于then(null, onRejected)）
  catch(onRejected) {
    return this.then(null, onRejected)
  }

  // finally方法（无论成功失败都会执行）
  finally(onFinally) {
    return this.then(
      value => MyPromise.resolve(onFinally()).then(() => value),
      reason => MyPromise.resolve(onFinally()).then(() => { throw reason })
    )
  }

  // 静态resolve方法
  static resolve(value) {
    if (value instanceof MyPromise) return value
    return new MyPromise(resolve => resolve(value))
  }

  // 静态reject方法
  static reject(reason) {
    return new MyPromise((_, reject) => reject(reason))
  }

  // 静态all方法（所有Promise成功才成功，有一个失败就失败）
  static all(promises) {
    return new MyPromise((resolve, reject) => {
      const results = []
      let count = 0
      const len = promises.length

      if (len === 0) return resolve(results)

      promises.forEach((promise, index) => {
        MyPromise.resolve(promise).then(
          value => {
            results[index] = value
            count++
            if (count === len) resolve(results)
          },
          reason => reject(reason)
        )
      })
    })
  }

  // 静态race方法（第一个完成的Promise决定结果）
  static race(promises) {
    return new MyPromise((resolve, reject) => {
      promises.forEach(promise => {
        MyPromise.resolve(promise).then(resolve, reject)
      })
    })
  }
}
```
