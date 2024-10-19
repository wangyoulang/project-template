<template>
    <div>
      <h2>后端数据展示</h2>
      <button @click="fetchHello">获取问候</button>
      <p>{{ message }}</p>
  
      <button @click="fetchUser">获取用户信息</button>
      <p v-if="userInfo">用户：{{ userInfo.name }}，年龄：{{ userInfo.age }}</p>
    </div>
  </template>
  
  <script>
  import axios from 'axios';
  
  export default {
    data() {
      return {
        message: '',
        userInfo: null,
      };
    },
    methods: {
      async fetchHello() {
        try {
          const response = await axios.get('http://127.0.0.1:8080/hello?name=lisi');
          this.message = response.data;
        } catch (error) {
          console.error('获取问候失败：', error);
        }
      },
      async fetchUser() {
        try {
          const response = await axios.get('http://127.0.0.1:8080/user');
          this.userInfo = response.data;
        } catch (error) {
          console.error('获取用户信息失败：', error);
        }
      },
    },
  };
  </script>
  
  <style scoped>
  button {
    margin: 5px;
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
  