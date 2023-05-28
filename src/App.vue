<template>
  <div class="calendar">
    <div class="header">
      <button @click="previousMonth" class="navigation-button">Previous</button>
      <h2 class="current-month">{{ currentMonth }}</h2>
      <button @click="nextMonth" class="navigation-button">Next</button>
    </div>
    <div class="calendar-grid">
      <div class="day-label">Monday</div>
      <div class="day-label">Tuesday</div>
      <div class="day-label">Wednesday</div>
      <div class="day-label">Thursday</div>
      <div class="day-label">Friday</div>
      <div class="day-label">Saturday</div>
      <div class="day-label">Sunday</div>
      <div class="day" v-for="day in days" :key="day">
        <div class="day-number" :class="{ 'current-day': isCurrentDay(day) }">{{ day }}</div>
        <div class="events">
          <div class="event" v-for="event in getEventsByDay(day)" :key="event.id">
            {{ event.title }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Calendar',
  data() {
    return {
      currentDate: new Date(),
      events: [],
    };
  },
  computed: {
    currentMonth() {
      return this.currentDate.toLocaleString('default', { month: 'long', year: 'numeric' });
    },
    days() {
      const days = [];
      const startDate = new Date(this.currentDate.getFullYear(), this.currentDate.getMonth(), 1);
      const endDate = new Date(this.currentDate.getFullYear(), this.currentDate.getMonth() + 1, 0);
      for (let i = startDate.getDate(); i <= endDate.getDate(); i++) {
        days.push(i);
      }
      return days;
    },
  },
  methods: {
    previousMonth() {
      this.currentDate = new Date(this.currentDate.getFullYear(), this.currentDate.getMonth() - 1, 1);
    },
    nextMonth() {
      this.currentDate = new Date(this.currentDate.getFullYear(), this.currentDate.getMonth() + 1, 1);
    },
    getEventsByDay(day) {
      const date = new Date(this.currentDate.getFullYear(), this.currentDate.getMonth(), day);
      return this.events.filter((event) => event.date.toDateString() === date.toDateString());
    },
    isCurrentDay(day) {
      const currentDate = new Date();
      const currentMonth = currentDate.getMonth();
      const currentYear = currentDate.getFullYear();

      return (
        this.currentDate.getDate() === day &&
        this.currentDate.getMonth() === currentMonth &&
        this.currentDate.getFullYear() === currentYear
      );
    },
  },
};
</script>

<style scoped>
body {
  background-color: #242424;
}

.calendar {
  font-family: 'Arial', sans-serif;
  max-width: 900px;
  margin: 0 auto;
  padding: 20px;
  background-color: #242424;
  color: white;
}

.header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 20px;
}

.navigation-button {
  background-color: #333;
  border: none;
  color: white;
  padding: 10px 20px;
  font-size: 16px;
  border-radius: 4px;
  cursor: pointer;
}

.current-month {
  font-size: 20px;
  font-weight: bold;
}

.calendar-grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 10px;
}

.day {
  background-color: #333;
  border-radius: 4px;
  padding: 20px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.day-label {
  font-size: 14px;
  font-weight: bold;
  margin-bottom: 5px;
}

.day-number {
  font-size: 18px;
  margin-bottom: 5px;
}

.events {
  margin-top: 5px;
}

.event {
  background-color: #555;
  padding: 5px;
  margin-bottom: 5px;
  border-radius: 4px;
  font-size: 14px;
}

.day.current-day {
  background-color: #ffcc99;
}
</style>
