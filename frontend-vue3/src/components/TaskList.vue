<template>
    <el-container class="task-app">
      <el-header class="header">
        <h2>任务管理</h2>
        <div class="summary">
          <div>待做任务：{{ pendingTasks.length }}</div>
          <div>已完成任务：{{ completedTasks.length }}</div>
        </div>
      </el-header>
  
      <el-main>
        <div class="add-task">
          <el-input
            v-model="newTask"
            placeholder="请输入任务名称，按回车保存"
            @keyup.enter="addTask"
            clearable
          />
        </div>
  
        <h3>待做任务</h3>
        <el-list class="task-list">
          <el-list-item
            v-for="(task, index) in pendingTasks"
            :key="task.id"
            class="task-item"
          >
            <div class="task-left">
              <div class="checkbox">
                <input type="checkbox" v-model="task.completed" @change="completeTask(task)" />
              </div>
  
              <button class="timer-button" @click="toggleTimer(task)">
                <span v-if="!task.isRunning">▶️</span>
                <span v-else>⏸️</span>
              </button>
            </div>
  
            <div class="task-body" @click="openPopover(task)">
              <el-popover ref="pendingPopover" width="160" trigger="click" placement="bottom">
                <el-button type="primary" size="mini" @click="startEditingTask(task)">编辑任务</el-button>
                <el-button type="danger" size="mini" @click="deleteTask(task.id)">删除任务</el-button>
                <template #reference>
                  <div v-if="!task.editing" class="task-name">{{ task.name }}</div>
                  <el-input
                    v-else
                    v-model="task.name"
                    ref="editInput"
                    @blur="finishEditingTask(task)"
                    @keyup.enter="finishEditingTask(task)"
                    size="small"
                  />
                </template>
              </el-popover>
            </div>
  
            <div class="timer-display">{{ formatTime(task.timer) }}</div>
          </el-list-item>
        </el-list>
  
        <h3>已完成任务</h3>
        <el-list class="task-list">
          <el-list-item
            v-for="(task, index) in completedTasks"
            :key="task.id"
            class="task-item completed-task"
          >
            <div class="task-left">
              <div class="checkbox">
                <input type="checkbox" disabled checked />
              </div>
            </div>
  
            <div class="task-body" @click="openPopover(task)">
              <el-popover ref="completedPopover" width="120" trigger="click" placement="bottom">
                <el-button type="danger" size="mini" @click="deleteTask(task.id)">删除任务</el-button>
                <template #reference>
                  <div class="completed-text">{{ task.name }}</div>
                </template>
              </el-popover>
            </div>
  
            <div class="timer-display">{{ formatTime(task.timer) }}</div>
          </el-list-item>
        </el-list>
      </el-main>
    </el-container>
  </template>
  
  <script>
  export default {
    data() {
      return {
        newTask: '',
        pendingTasks: this.loadTasks('pending'),
        completedTasks: this.loadTasks('completed'),
      };
    },
    methods: {
      addTask() {
        if (this.newTask.trim() !== '') {
          const task = {
            id: Date.now(),
            name: this.newTask.trim(),
            timer: 0,
            isRunning: false,
            intervalId: null,
            completed: false,
            editing: false,
          };
          this.pendingTasks.push(task);
          this.newTask = '';
          this.saveTasks();
        }
      },
      completeTask(task) {
        this.stopTimer(task);
        task.completed = true;
        this.completedTasks.push(task);
        this.pendingTasks = this.pendingTasks.filter(t => t.id !== task.id);
        this.saveTasks();
      },
      toggleTimer(task) {
        if (task.isRunning) {
          this.pauseTimer(task);
        } else {
          this.startTimer(task);
        }
      },
      startTimer(task) {
        this.pendingTasks.forEach(t => this.pauseTimer(t));
        task.isRunning = true;
        task.intervalId = setInterval(() => {
          task.timer += 1;
          this.saveTasks();
        }, 1000);
      },
      pauseTimer(task) {
        this.stopTimer(task);
        task.isRunning = false;
      },
      stopTimer(task) {
        if (task.intervalId) {
          clearInterval(task.intervalId);
          task.intervalId = null;
        }
      },
      startEditingTask(task) {
        task.editing = true;
        this.$nextTick(() => {
          this.$refs.editInput.focus();
        });
      },
      finishEditingTask(task) {
        task.editing = false;
        this.saveTasks();
      },
      deleteTask(id) {
        this.pendingTasks = this.pendingTasks.filter(task => task.id !== id);
        this.completedTasks = this.completedTasks.filter(task => task.id !== id);
        this.saveTasks();
      },
      openPopover(task) {
        const popover = this.$refs.pendingPopover;
        popover.doShow();
      },
      saveTasks() {
        localStorage.setItem('pendingTasks', JSON.stringify(this.pendingTasks));
        localStorage.setItem('completedTasks', JSON.stringify(this.completedTasks));
      },
      loadTasks(listType) {
        const tasks = localStorage.getItem(listType === 'pending' ? 'pendingTasks' : 'completedTasks');
        return tasks ? JSON.parse(tasks) : [];
      },
      formatTime(seconds) {
        const hours = String(Math.floor(seconds / 3600)).padStart(2, '0');
        const minutes = String(Math.floor((seconds % 3600) / 60)).padStart(2, '0');
        const secs = String(seconds % 60).padStart(2, '0');
        return `${hours}:${minutes}:${secs}`;
      },
    },
    beforeUnmount() {
      this.pendingTasks.forEach(task => this.stopTimer(task));
    },
  };
  </script>
  
  <style scoped>
  .task-app {
  height: 100vh;
  display: flex;
  flex-direction: column;
  background-color: #f7f8fa;
  font-family: 'Roboto', sans-serif;
}

.header {
  padding: 16px;
  background-color: #fff;
  border-bottom: 1px solid #e5e5e5;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.add-task {
  margin: 20px;
}

.task-list {
  margin-top: 10px;
  padding: 0 20px;
}

.task-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background-color: #fff;
  border-radius: 8px;
  padding: 12px 16px;
  margin-bottom: 10px;
  box-shadow: 0px 1px 3px rgba(0, 0, 0, 0.1);
  transition: box-shadow 0.3s ease;
}

.task-item:hover {
  box-shadow: 0px 2px 6px rgba(0, 0, 0, 0.15);
}

.task-left {
  display: flex;
  align-items: center;
  flex: 0;
  margin-right: 10px;
}

.circle-checkbox {
  width: 24px;
  height: 24px;
  border: 2px solid #d3d3d3;
  border-radius: 50%;
  margin-right: 10px;
  cursor: pointer;
  transition: border-color 0.2s;
}

.circle-checkbox input:checked + & {
  background-color: #4caf50;
  border-color: #4caf50;
}

.timer-button {
  background: none;
  border: none;
  font-size: 20px;
  margin-right: 10px;
  cursor: pointer;
  color: #4a90e2;
}

.task-name-container {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: flex-start; /* 确保任务名称左对齐 */
}
.task-body {
  flex: 1;
  display: flex;
  align-items: center;  /* 垂直居中 */
  justify-content: flex-start;  /* 水平左对齐 */
  cursor: pointer;
}

.task-name {
  font-size: 16px;
  font-weight: 500;
  color: #333;
  text-align: left;
  cursor: pointer;
  transition: color 0.2s;
  width: 100%; /* 让名称占满容器 */
}

.task-name:hover {
  color: #4a90e2;
}

.completed-text {
  text-decoration: line-through;
  color: #aaa;
}

.timer-display {
  font-family: 'Courier New', monospace;
  font-size: 16px;
  text-align: right;
  width: 80px;
  color: #555;
}

.completed-task .timer-display {
  color: #4caf50;
}

.el-button {
  margin: 10px;
  /* width: 100%; 让按钮占据父容器的全部宽度 */
  text-align: center; /* 确保按钮文本居中 */
}

.el-popover {
  border-radius: 8px;
  box-shadow: 0px 2px 10px rgba(0, 0, 0, 0.1);
}
  </style>
  