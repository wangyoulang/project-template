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
  
          <span class="task-text">{{ task.name }}</span>
  
          <div class="timer-display">
            <span>{{ formatTime(task.timer) }}</span>
          </div>
  
          <el-button
            class="action-button"
            @click="toggleTimer(task)"
            size="small"
          >
            {{ task.isRunning ? 'Pause' : 'Start' }}
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
          <el-checkbox v-model="task.completed" disabled />
          <span class="task-text">{{ task.name }}</span>
  
          <div class="timer-display">
            <span>Time spent: {{ formatTime(task.timer) }}</span>
          </div>
        </el-list-item>
      </el-list>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        newTask: '', // 输入框的内容
        pendingTasks: [], // 待做任务列表
        completedTasks: [], // 已完成任务列表
      };
    },
    methods: {
      addTask() {
        if (this.newTask.trim() !== '') {
          const task = {
            id: Date.now(), // 使用时间戳作为唯一 ID
            name: this.newTask,
            timer: 0, // 计时器秒数
            intervalId: null, // 计时器 ID
            isRunning: false, // 是否在计时
            completed: false, // 是否完成
          };
          this.pendingTasks.push(task);
          this.newTask = ''; // 清空输入框
        }
      },
  
      completeTask(index) {
        const completedTask = this.pendingTasks.splice(index, 1)[0];
        this.stopTimer(completedTask); // 停止计时器
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
        // 停止其他所有任务的计时
        this.pendingTasks.forEach(t => {
          if (t !== task) {
            this.pauseTimer(t);
          }
        });
  
        // 启动当前任务的计时
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
            const time = Math.floor(remaining /
            unit.value);
          remaining %= unit.value;
          result += `${time}${unit.label} `;
        }
      }

      return result.trim() || '0s';
    },
  },
  beforeUnmount() {
    // 清理所有任务的计时器
    this.pendingTasks.forEach(task => this.stopTimer(task));
  },
};
</script>

<style scoped>
.task-app {
  width: 500px;
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
  width: 100px;
  text-align: center;
  border: 1px solid #ddd;
  border-radius: 4px;
  padding: 5px;
  margin-left: 10px;
  background-color: #f9f9f9;
}

.action-button {
  width: 80px;
  text-align: center;
}

.el-button {
  margin-left: 10px;
}
</style>
