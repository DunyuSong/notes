# Vue学习笔记
## 常用指令
### v-for
1. 遍历数组值，并且将数组中每一项值传递给子组件
```html
    <ul class="list-group">
      <!-- 遍历Item对象，v-for指令第一个参数为属性值，第二个参数为属性名，第三个参数为index。
      可以将遍历的数据项通过属性的方式传递给子组件，将属性名为comment1传递给Item组件 -->
      <!-- <Item v-for="(commentValue, commentVame, index) in comments" :key="index" :comment1="commentValue"/> -->
      <!-- v-for第二种简化的写法 -->
      <Item v-for="(comment, index) in comments" :key="index" :comment1="comment"/>
    </ul>
```

## Vue中的组件
1. 组件是一个功能界面的组合，一般情况下包含三部分。HTML、css、JavaScript、静态资源
2. 在Vue中一个组件包含三部分
```html
<template>
    <div>
    </div>
</template>
<script>
    export default {}
</script>
<style>
</style>

```
