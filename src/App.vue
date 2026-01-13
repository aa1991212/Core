<script>
export default {
  data() {
    return {
      theme: 'day',      
      newTask: '',       
      newDate: '',
      // newPriority: 'normal', // 這行刪除，因為變自動了
      
      tasks: [           
        { name: 'a', date: '2026-01-11', priority: 'high' },
        { name: 'b', date: '2026-01-18', priority: 'normal' }
      ],
      historyTasks: [],
      player: {
        level: 42,      
        currentXP: 75,  
        maxXP: 100      
      },
      hourDeg: 0, minDeg: 0, secDeg: 0, timer: null,
      currentDate: new Date(), 
    }
  },
  computed: {
    currentYear() { return this.currentDate.getFullYear(); },
    currentMonth() { return this.currentDate.getMonth(); },
    calendarGrid() {
      const year = this.currentYear;
      const month = this.currentMonth;
      let firstDay = new Date(year, month, 1).getDay();
      firstDay = firstDay === 0 ? 7 : firstDay; 
      const daysInMonth = new Date(year, month + 1, 0).getDate();
      const prevMonthDays = new Date(year, month, 0).getDate();
      const grid = [];
      for (let i = 1; i < firstDay; i++) { grid.push({ num: prevMonthDays - (firstDay - 1 - i), isOtherMonth: true }); }
      for (let i = 1; i <= daysInMonth; i++) { grid.push({ num: i, isOtherMonth: false, fullDate: new Date(year, month, i).toDateString() }); }
      const remainingCells = 42 - grid.length;
      for (let i = 1; i <= remainingCells; i++) { grid.push({ num: i, isOtherMonth: true }); }
      return grid;
    }
  },
  methods: {
    // --- 自動判定邏輯 (修改處) ---
    addTask() {
      if (!this.newTask) return;

      // 預設一般
      let autoPriority = 'normal';

      // 如果有填日期，就進行計算
      if (this.newDate) {
        const today = new Date();
        today.setHours(0, 0, 0, 0); // 歸零時間，只比對日期

        const targetDate = new Date(this.newDate);
        targetDate.setHours(0, 0, 0, 0);

        // 計算差距毫秒數 -> 轉成天數
        const diffTime = targetDate - today;
        const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24));

        // 規則：如果是 過去(過期) 或 未來3天內 -> 設定為緊急
        if (diffDays <= 3) {
          autoPriority = 'high';
        }
      }

      this.tasks.push({
        name: this.newTask,
        date: this.newDate || '未定', 
        priority: autoPriority // 使用自動計算的結果
      });

      this.newTask = ''; 
      this.newDate = '';
    },

    completeTask(index) {
      const task = this.tasks[index];
      this.tasks.splice(index, 1);
      this.historyTasks.unshift(task);
      this.addExp(20);
    },
    undoTask(index) {
      const task = this.historyTasks[index];
      this.historyTasks.splice(index, 1);
      this.tasks.push(task);
      this.addExp(-20);
    },
    deleteTask(index) {
      this.tasks.splice(index, 1);
    },
    addExp(amount) {
      this.player.currentXP += amount;
      while (this.player.currentXP >= this.player.maxXP) {
        this.player.currentXP -= this.player.maxXP;
        this.player.level++;
      }
      while (this.player.currentXP < 0) {
        if (this.player.level > 1) {
          this.player.level--;
          this.player.currentXP += this.player.maxXP;
        } else {
          this.player.currentXP = 0; break;
        }
      }
    },
    toggleTheme() { this.theme = this.theme === 'day' ? 'night' : 'day'; },
    updateTime() {
      const now = new Date();
      this.hourDeg = (now.getHours() % 12) * 30 + now.getMinutes() * 0.5;
      this.minDeg = now.getMinutes() * 6;
      this.secDeg = now.getSeconds() * 6;
    },
    changeMonth(offset) {
      const newDate = new Date(this.currentDate);
      newDate.setMonth(newDate.getMonth() + offset);
      this.currentDate = newDate;
    },
    isToday(day) {
      if (day.isOtherMonth) return false;
      return day.fullDate === new Date().toDateString();
    }
  },
  mounted() { this.updateTime(); this.timer = setInterval(this.updateTime, 1000); },
  beforeUnmount() { if (this.timer) clearInterval(this.timer); }
}
</script>

<template>
  <div class="page" :class="theme">
    <div class="dashboard-container">
      
      <div class="main-panel">
        <h1 class="glitch-header">任務中心</h1>
        
        <div class="progress-section">
          <div class="progress-info">
            <span>Operator Lv.{{ player.level }}</span>
            <span class="xp-text">{{ player.currentXP }} / {{ player.maxXP }} XP</span>
          </div>
          <div class="progress-bar">
            <div class="progress-fill" :style="{ width: (player.currentXP / player.maxXP * 100) + '%' }"></div>
          </div>
        </div>

        <div class="command-console">
          <div class="input-group">
            <input type="text" v-model="newTask" placeholder="輸入作戰代碼..." class="cyber-input text-input" @keyup.enter="addTask">
            <input type="date" v-model="newDate" class="cyber-input date-input">
            <button @click="addTask" class="cyber-btn">寫入系統</button>
          </div>
        </div>

        <div class="task-container">
          <div class="list-section" v-if="tasks.length > 0">
            <div class="section-title">>> 執行中任務</div>
            <div class="item" v-for="(task, index) in tasks" :key="'task-'+index">
              <div class="status-indicator" :class="task.priority"></div>
              <div class="item-content">
                <div class="item-header">
                  <span class="badge" :class="task.priority">{{ task.priority === 'high' ? '緊急' : '一般' }}</span>
                  <span class="task-date">{{ task.date }}</span>
                </div>
                <div class="name">{{ task.name }}</div>
              </div>
              <div class="btn-group">
                <button class="icon-btn btn-check" @click="completeTask(index)">✓</button>
                <button class="icon-btn btn-delete" @click="deleteTask(index)">×</button>
              </div>
            </div>
          </div>

          <div class="list-section" v-if="historyTasks.length > 0">
            <div class="section-title">>> 歷史紀錄 (已完成)</div>
            <div class="item history" v-for="(task, index) in historyTasks" :key="'hist-'+index">
              <div class="item-content">
                <div class="item-header">
                  <span class="badge" :class="task.priority">{{ task.priority === 'high' ? '緊急' : '一般' }}</span>
                </div>
                <div class="name">{{ task.name }}</div>
              </div>
              <div class="btn-group">
                <button class="icon-btn btn-undo" @click="undoTask(index)">↺</button>
              </div>
            </div>
          </div>

          <div class="empty-msg" v-if="tasks.length === 0 && historyTasks.length === 0">>> 等待指令輸入...</div>
        </div>
      </div>

      <div class="side-panel">
        <div class="calendar-card">
          <div class="cal-header">
             <span class="nav-btn" @click="changeMonth(-1)">&#10094;</span>
             {{ currentYear }}年 {{ currentMonth + 1 }}月
             <span class="nav-btn" @click="changeMonth(1)">&#10096;</span>
          </div>
          <div class="cal-grid">
            <div class="cal-day-name">一</div><div class="cal-day-name">二</div><div class="cal-day-name">三</div><div class="cal-day-name">四</div><div class="cal-day-name">五</div><div class="cal-day-name">六</div><div class="cal-day-name">日</div>
            <div class="cal-day" v-for="(day, index) in calendarGrid" :key="index" :class="{ 'other-month': day.isOtherMonth, 'today': isToday(day) }">{{ day.num }}</div>
          </div>
        </div>
        <div class="tactical-watch-card" @click="toggleTheme">
          <div class="clock-title">場景時鐘 // <span :style="{color: theme === 'day' ? '#333' : '#ef4444'}">運行中</span></div>
          <div class="watch-face">
            <div class="center-nut"></div>
            <div class="marker m-12"></div><div class="marker m-3"></div><div class="marker m-6"></div><div class="marker m-9"></div>
            <div class="hand hour-hand" :style="{ transform: `rotate(${hourDeg}deg)` }"></div>
            <div class="hand min-hand" :style="{ transform: `rotate(${minDeg}deg)` }"></div>
            <div class="hand sec-hand" :style="{ transform: `rotate(${secDeg}deg)` }"></div>
          </div>
          <p class="click-hint">>> 點擊切換場景</p>
        </div>
      </div>
    </div>
  </div>
</template>

<style src="./style.css"></style>