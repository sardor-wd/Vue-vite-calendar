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

      <div class="day" v-for="day in days" :key="day" @dragover="onDragOver" @drop="onDrop($event, day)">
        <div class="day-number" :class="{ 'current-day': isCurrentDay(day) }" @dblclick="openAddPopup(day)"
             @dragend="onDragEnd">
          {{ day }}
        </div>
        <div class="events" @dragover="onDragOver" @drop="onDrop($event, day)">
          <div class="event" v-for="event in getEventsByDay(day)" :key="event.id" draggable="true"
               @dragstart="onDragStart($event)" :data-id="event.id">
            <span class="event-title">{{ event.title }}</span>
            <button class="delete-button" @click="openDeletePopup(event.id)">Delete</button>
          </div>
        </div>
      </div>
    </div>
    <div class="add-event">
      <button @click="openAddPopup" class="add-button">Add Event</button>
    </div>
    <div v-if="showAddPopup" class="popup">
      <div class="popup-content">
        <h3>Add Event</h3>
        <input type="text" v-model="newEventTitle" placeholder="Event title"/>
        <input type="date" v-model="newEventDate"/>
        <div class="popup-buttons">
          <button @click="addEvent" class="confirm-button">Add</button>
          <button @click="closeAddPopup" class="cancel-button">Cancel</button>
        </div>
      </div>
    </div>
    <div v-if="showDeletePopup && selectedEventId !== null" class="popup">
      <div class="popup-content">
        <h3>Delete Event</h3>
        <p>Are you sure you want to delete this event?</p>
        <div class="popup-buttons">
          <button @click="deleteEvent(selectedEventId)" class="confirm-button">Delete</button>
          <button @click="closeDeletePopup" class="cancel-button">Cancel</button>
        </div>
      </div>
    </div>
  </div>
</template>


<script>
import axios from 'axios';
import moment from 'moment';

export default {
  name: 'Calendar',
  data() {
    return {
      currentDate: new Date(),
      events: [],
      newEventTitle: '',
      newEventDate: '',
      showAddPopup: false,
      showDeletePopup: false,
      selectedEventId: null,
      isDragging: false,
      draggedEventId: null,
      apiUrl: 'http://127.0.0.1:8000/api/events/',
      deleteUrl: 'http://127.0.0.1:8000/api/events/delete/',
      createUrl: 'http://127.0.0.1:8000/api/events/create/',
      updateUrl: 'http://127.0.0.1:8000/api/events/update/'
    };
  },
  computed: {
    currentMonth() {
      return this.currentDate.toLocaleString('default', {
        month: 'long',
        year: 'numeric'
      });
    },
    days() {
      const days = [];
      const startDate = new Date(
          this.currentDate.getFullYear(),
          this.currentDate.getMonth(),
          1
      );
      const endDate = new Date(
          this.currentDate.getFullYear(),
          this.currentDate.getMonth() + 1,
          0
      );
      for (let day = startDate.getDate(); day <= endDate.getDate(); day++) {
        days.push(day);
      }
      return days;
    }
  },
  mounted() {
    this.fetchEvents();
  },
  methods: {
    onDragStart(event) {
      const eventId = event.target.getAttribute('data-id');
      console.log('Dragging event:', eventId);
      event.dataTransfer.setData('text/plain', eventId);
    },
    onDrop(event, day) {
      event.preventDefault();
      const eventId = event.dataTransfer.getData('text/plain');
      const eventObj = this.events.find(event => event.id === eventId);
      if (eventObj) {
        eventObj.date = new Date(
            this.currentDate.getFullYear(),
            this.currentDate.getMonth(),
            day
        );
        this.updateEvent(eventObj); // Call the updateEvent method
      }
    },
    onDragOver(event) {
      event.preventDefault();
    },

    onDragEnd() {
      this.isDragging = false;
    },
    updateEvent(event) {
      const updatedEvent = {
        id: event.id,
        title: event.title,
        date: moment(event.date).format('YYYY-MM-DD')
      };
      console.log(updatedEvent);

      axios
          .put(`${this.updateUrl}${event.id}/`, updatedEvent)
          .then(response => {
            console.log(response.data.message);
            this.fetchEvents(); // Fetch updated events after successful update
          })
          .catch(error => {
            console.error(error);
          });
    },
    fetchEvents() {
      axios
          .get(this.apiUrl)
          .then(response => {
            this.events = response.data.map(event => ({
              id: event.id,
              title: event.title,
              description: event.description,
              date: moment(event.date).format('YYYY-MM-DD')
            }));
          })
          .catch(error => {
            console.error('Error fetching events:', error);
          });
    },
    isCurrentDay(day) {
      const today = new Date();
      return (
          this.currentDate.getFullYear() === today.getFullYear() &&
          this.currentDate.getMonth() === today.getMonth() &&
          day === today.getDate()
      );
    },
    previousMonth() {
      this.currentDate.setMonth(this.currentDate.getMonth() - 1);
      this.fetchEvents();
    },
    nextMonth() {
      this.currentDate.setMonth(this.currentDate.getMonth() + 1);
      this.fetchEvents();
    },
    openAddPopup(day) {
      this.showAddPopup = true;
    },
    closeAddPopup() {
      this.showAddPopup = false;
      this.newEventTitle = '';
      this.newEventDate = '';
    },
    addEvent() {
      const newEvent = {
        title: this.newEventTitle,
        date: this.newEventDate
      };
      axios
          .post(this.createUrl, newEvent)
          .then((response) => {
            console.log(response.data.message);
            this.closeAddPopup();
            this.fetchEvents();
          })
          .catch((error) => {
            console.error(error);
          });
    },
    deleteEvent(eventId) {
      axios
          .delete(`${this.deleteUrl}${eventId}`)
          .then((response) => {
            console.log(response.data.message);
            this.closeDeletePopup();
            this.fetchEvents();
          })
          .catch((error) => {
            console.error(error);
          });
    },
    openDeletePopup(eventId) {
      this.selectedEventId = eventId;
      this.showDeletePopup = true;
    },
    closeDeletePopup() {
      this.selectedEventId = null;
      this.showDeletePopup = false;
    },
    getEventsByDay(day) {
      return this.events.filter((event) => {
        const eventDate = new Date(event.date);
        return (
            eventDate.getDate() === day &&
            eventDate.getMonth() === this.currentDate.getMonth() &&
            eventDate.getFullYear() === this.currentDate.getFullYear()
        );
      })
    },
  }
};
</script>

<style scoped>
.calendar {
  font-family: "Arial", sans-serif;
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
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
}

.day {
  background-color: #333;
  border-radius: 4px;
  padding: 20px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  cursor: pointer;
  transition: background-color 0.3s;
}

.day:hover {
  background-color: #444;
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

.add-event {
  margin-top: 20px;
}

.add-button {
  background-color: #00bfa5;
  border: none;
  color: white;
  padding: 10px 20px;
  font-size: 16px;
  border-radius: 4px;
  cursor: pointer;
}

.delete-button {
  background-color: #dc3545;
  border: none;
  color: white;
  border-radius: 4px;
  cursor: pointer;
}

.popup {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: rgba(0, 0, 0, 0.5);
}

.popup-content {
  display: flex;
  flex-direction: column;
  background-color: #242424;
  padding: 20px;
  border-radius: 4px;
  text-align: center;
}

.popup-content input[type="text"],
.popup-content input[type="date"] {
  margin-bottom: 10px;
  padding: 10px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 4px;
  background-color: #242424;
  color: white;
}

.popup h3 {
  margin-top: 0;
}

.popup-buttons {
  margin-top: 20px;
}

.confirm-button {
  background-color: #00bfa5;
  border: none;
  color: white;
  padding: 10px 20px;
  font-size: 16px;
  border-radius: 4px;
  cursor: pointer;
  margin-right: 10px;
}

.cancel-button {
  background-color: #333;
  border: none;
  color: white;
  padding: 10px 20px;
  font-size: 16px;
  border-radius: 4px;
  cursor: pointer;
}

.current-day {
  width: 30px;
  height: 30px;
  border-radius: 50%;
  background-color: #00bfa5;
  display: flex;
  justify-content: center;
  align-items: center;
  color: white;
}

@media (min-width: 768px) {
  .calendar-grid {
    grid-template-columns: repeat(7, 1fr);
  }
}

@media (max-width: 768px) {
  .day-label {
    display: none;
  }
}
</style>