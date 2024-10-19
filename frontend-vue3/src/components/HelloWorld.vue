<!-- src/components/HelloWorld.vue -->
<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <button @click="printMessage">点击我</button>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  props: {
    msg: String, // 接收父组件传递的 msg 属性
  },
  methods: {
    printMessage() {
      console.log("按钮被点击了！");
    }
  },
};
</script>

<style scoped>
button {
  margin-top: 10px;
  padding: 10px 20px;
  background-color: #42b983;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
button:hover {
  background-color: #369d7e;
}
</style>
