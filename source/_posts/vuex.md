---
title: Vuex和pinia
---

### vuex的几种状态

#### 1. state
- state是vuex存储的响应式状态,是数据的唯一来源,每个vuex应用只有一个state对象

#### 2. getters
- getters是vuex的计算属性,用于派生state,getter的返回值会根据它的依赖被缓存起来,只有当它的依赖值发生了改变才会被重新计算

#### 3. mutations
- mutations是vuex的提交函数,用于修改state

#### 4. actions
- actions是vuex的异步操作,可以包含任意异步操作,可以触发mutations来修改state

#### 5. modules
- modules是vuex的模块化方案,允许我们将store分割成模块,每个模块拥有自己的state、getters、mutations、actions

#### 模板代码

```javascript
import Vue from 'vue';
import Vuex from 'vuex';
Vue.use(Vuex);
export default new Vuex.Store({
    state: {
        count:0,
    },
    mutations:{
        increment(state){
            state.count++;
        },
        decrement(state){
            state.count--;
        }
    },
    actions:{
        increment({commit}){
            commit('increment');
        },
        decrement({commit}){
            commit('decrement');
        }
    },
    getters:{
        count:state=>state.count //使用一个回调,把state作为参数,返回count的值,实时更新state参数count的值
    }
})
```
### pinia的几种状态

#### 代码

```javascript
// store.js
import { defineStore } from 'pinia';
import { ref, computed } from 'vue';

export const useCounterStore = defineStore('counter', () => {
  const count = ref(0);

  const increment = () => {
    count.value++;
  };

  const decrement = () => {
    count.value--;
  };

  const doubleCount = computed(() => count.value * 2);

  return { count, increment, decrement, doubleCount };
});

```