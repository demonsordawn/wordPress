---
title: v-model和v-bind的区别
tags: [vue]
---

### v-bind
1. 单向绑定:输出从js流向DOM
2. 语法:v-bind:属性名='数据',或者简写为: :属性名='数据'
3. 作用:将数据绑定到HTML属性,或者props

### v-model
1. 双向绑定:数据在DOM和js组件双向同步(双向绑定),当数据发生变化时,DOM会同步更新,js组件也会同步更新
2. 语法:v-model='数据'
3. 作用:主要作为表单输入元素,实现数据和视图的双向同步
4. 原理:
```html
//v-model实际上是语法糖
// 这两种写法是等价的
<input type="text" v-model="message">

<input type='text' :value='message' @input='message=$event.target.value'/>
```
5. 使用v-model,实现组件prop的传递
```javascript
<!-- 父组件 -->
<template>
  <CustomInput v-model="username" />
  
  <!-- 相当于 -->
  <CustomInput 
    :modelValue="username" 
    @update:modelValue="newValue => username = newValue" 
  />
</template>

<script setup>
import { ref } from 'vue'

const username = ref('')
</script>

<!-- 子组件 CustomInput.vue -->
<template>
  <input 
    :value="modelValue" 
    @input="$emit('update:modelValue', $event.target.value)"
  >
</template>

<script setup>
defineProps(['modelValue'])
defineEmits(['update:modelValue'])
</script>


<!-- vue3支持多个model的绑定,要标明model的名称 -->
 <!-- 父组件 -->
<template>
  <UserForm 
    v-model:name="userName" 
    v-model:email="userEmail" 
  />
</template>

<script setup>
import { ref } from 'vue'

const userName = ref('')
const userEmail = ref('')
</script>

<!-- 子组件 UserForm.vue -->
<template>
  <input 
    :value="name" 
    @input="$emit('update:name', $event.target.value)"
  >
  <input 
    :value="email" 
    @input="$emit('update:email', $event.target.value)"
  >
</template>

<script setup>
defineProps(['name', 'email'])
defineEmits(['update:name', 'update:email'])
</script>
```
6. $emit 是 Vue 中用于子组件向父组件通信的方法，它允许子组件触发自定义事件，父组件可以监听这些事件并做出响应。