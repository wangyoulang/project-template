<template>
    <div class="task-app">
      <h2>任务清单</h2>
      <el-input
        v-model="newTask"
        placeholder="请输入任务并按回车"
        @keyup.enter="addTask"
        clearable
      />
  
      <h3>待做任务</h3>
      <el-list>
        <el-list-item
          v-for="(task, index) in pendingTasks"
          :key="task.id"
          class="task-item"
        >
          <el-checkbox @change="completeTask(index)" />
  
          <!-- 编辑或显示任务名称 -->
          <span
            v-if="!task.editing"
            class="task-text"
            @dblclick="editTask(task)"
          >
            {{ task.name }}
          </span>
          <el-input
            v-else
            v-model="task.name"
            size="small"
            @blur="saveTask(task)"
            @keyup.enter="saveTask(task)"
            @keyup.esc="cancelEdit(task)"
          />
  
          <div class="timer-display">
            <el-tag type="info" class="timer-tag">
              {{ formatTime(task.timer) }}
            </el-tag>
          </div>
  
          <el-button
            class="action-button"
            :type="task.isRunning ? 'danger' : 'success'"
            icon="el-icon-time"
            @click="toggleTimer(task)"
            size="small"
          >
            {{ task.isRunning ? 'Pause' : 'Start' }}
          </el-button>
  
          <el-button
            class="delete-button"
            type="danger"
            icon="el-icon-delete"
            @click="deleteTask(index, 'pending')"
            size="small"
          >
            Delete
          </el-button>
        </el-list-item>
      </el-list>
  
      <h3>已完成任务</h3>
      <el-list>
        <el-list-item
          v-for="(task, index) in completedTasks"
          :key="task.id"
          class="task-item"
        >
          <span class="task-text">{{ task.name }}</span>
  
          <div class="timer-display">
            <el-tag type="success" effect="dark">
              Time spent: {{ formatTime(task.timer) }}
            </el-tag>
          </div>
  
          <el-button
            class="delete-button"
            type="danger"
            icon="el-icon-delete"
            @click="deleteTask(index, 'completed')"
            size="small"
          >
            Delete
          </el-button>
        </el-list-item>
      </el-list>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        newTask: '',
        pendingTasks: [],
        completedTasks: [],
      };
    },
    methods: {
      addTask() {
        if (this.newTask.trim() !== '') {
          const task = {
            id: Date.now(),
            name: this.newTask,
            timer: 0,
            intervalId: null,
            isRunning: false,
            completed: false,
            editing: false,
          };
          this.pendingTasks.push(task);
          this.newTask = '';
        }
      },
      completeTask(index) {
        const completedTask = this.pendingTasks.splice(index, 1)[0];
        this.stopTimer(completedTask);
        completedTask.completed = true;
        this.completedTasks.push(completedTask);
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
      editTask(task) {
        task.editing = true;
      },
      saveTask(task) {
        task.editing = false;
      },
      cancelEdit(task) {
        task.editing = false;
      },
      deleteTask(index, listType) {
        if (listType === 'pending') {
          this.pendingTasks.splice(index, 1);
        } else if (listType === 'completed') {
          this.completedTasks.splice(index, 1);
        }
      },
      formatTime(seconds) {
        const units = [
          { label: 'w', value: 604800 },
          { label: 'd', value: 86400 },
          { label: 'h', value: 3600 },
          { label: 'm', value: 60 },
          { label: 's', value: 1 },
        ];
        let remaining = seconds;
        let result = '';
        for (const unit of units) {
          if (remaining >= unit.value) {
            const time = Math.floor(remaining / unit.value);
            remaining %= unit.value;
            result += `${time}${unit.label} `;
          }
        }
        return result.trim() || '0s';
      },
    },
    beforeUnmount() {
      this.pendingTasks.forEach(task => this.stopTimer(task));
    },
  };
  </script>
  
  <style scoped>
  .task-app {
    width: 600px;
    margin: 0 auto;
    text-align: left;
    font-family: Arial, sans-serif;
  }
  
  .task-item {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 10px;
  }
  
  .task-text {
    flex: 1;
    margin-left: 10px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }
  
  .timer-display {
    width: 120px;
    margin-left: 10px;
    display: flex;
    justify-content: center;
  }
  
  .timer-tag {
    width: 100%;
    text-align: center;
  }
  
  .action-button {
    width: 60px;
    text-align: center;
    margin-left: 5px;
  }
  
  .delete-button {
    margin-left: 10px;
  }
  </style>
  