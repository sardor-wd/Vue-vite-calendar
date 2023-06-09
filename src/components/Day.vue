<template>
  <div class="calendar" style="margin-top: 50px">
    <div class="header">
      <button @click="previousDay" class="navigation-button">Previous</button>
      <h2 class="current-day">{{ getCurrentDayFormatted() }}</h2>
      <button @click="nextDay" class="navigation-button">Next</button>
    </div>
    <div class="timetable" :class="{ 'scrollable': shouldScroll }">
      <div class="time-label">Time</div>
      <div class="time-slot" v-for="timeSlot in timeSlots" :key="timeSlot" @dragover="onDragOver($event)"
           @drop="onDrop($event, timeSlot)" @dblclick="openAddPopup(timeSlot)">
        <div class="time">{{ timeSlot }}</div>
        <div class="event-container">
          <div class="event" v-for="event in getEventsByTime(timeSlot)" :key="event.id" draggable="true"
               @dragstart="onDragStart($event, event)" :data-id="event.id">
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
        <input type="time" v-model="newEventTime"/>
        <p>Date: {{ getCurrentDayFormatted() }}</p>
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
import axios from "axios";

export default {
  data() {
    return {
      currentDay: new Date(),
      timeSlots: [],
      events: [],
      newEventTitle: "",
      newEventTime: "",
      showAddPopup: false,
      showDeletePopup: false,
      selectedEventId: null,
      shouldScroll: false,
      apiUrl: "http://127.0.0.1:8000/api/events/",
      deleteUrl: "http://127.0.0.1:8000/api/events/delete/",
      createUrl: "http://127.0.0.1:8000/api/events/create/",
      updateUrl: "http://127.0.0.1:8000/api/events/update/",
    };
  },
  created() {
    const startTime = new Date();
    startTime.setHours(0, 0, 0, 0);

    for (let i = 0; i < 96; i++) {
      const time = new Date(startTime.getTime() + i * 15 * 60 * 1000);
      const hours = String(time.getHours()).padStart(2, "0");
      const minutes = String(time.getMinutes()).padStart(2, "0");
      this.timeSlots.push(`${hours}:${minutes}`);
    }
  },
  methods: {
    getCurrentDayFormatted() {
      const options = {weekday: "long", day: "numeric"};
      return this.currentDay.toLocaleDateString(undefined, options);
    },
    previousDay() {
      const previousDay = new Date(this.currentDay);
      previousDay.setDate(previousDay.getDate() - 1);
      this.currentDay = previousDay;
    },
    nextDay() {
      const nextDay = new Date(this.currentDay);
      nextDay.setDate(nextDay.getDate() + 1);
      this.currentDay = nextDay;
    },
    getEventsByTime(timeSlot) {
      return this.events.filter((event) => event.time === timeSlot);
    },
    onDragOver(event) {
      event.preventDefault();
    },
    onDragStart(event, eventObj) {
      event.dataTransfer.setData("text/plain", JSON.stringify(eventObj));
      event.dataTransfer.setData("timeSlot", eventObj.time);
    },
    onDrop(event, timeSlot) {
      event.preventDefault();
      const eventData = event.dataTransfer.getData("text/plain");
      const eventObj = JSON.parse(eventData);

      if (eventObj) {
        eventObj.time = timeSlot;
        this.updateEvent(eventObj);
      }
      console.log(eventObj);
    },
    openDeletePopup(eventId) {
      this.selectedEventId = eventId;
      this.showDeletePopup = true;
    },
    addEvent() {
      const newEvent = {
        title: this.newEventTitle,
        time: this.newEventTime,
        date: this.currentDay.toISOString().split("T")[0],
      };
      axios
          .post(this.createUrl, newEvent)
          .then((response) => {
            this.events.push(response.data);
            this.checkScroll();
            this.closeAddPopup();
          })
          .catch((error) => {
            console.error('Error creating event:', error);
          });
    },

    deleteEvent(eventId) {
      const deleteUrl = this.deleteUrl + eventId;
      axios
          .delete(deleteUrl)
          .then(() => {
            this.events = this.events.filter((event) => event.id !== eventId);
            this.checkScroll();
            this.closeDeletePopup();
          })
          .catch((error) => {
            console.error('Error deleting event:', error);
          });
    },
    closeDeletePopup() {
      this.selectedEventId = null;
      this.showDeletePopup = false;
    },
    openAddPopup() {
      this.showAddPopup = true;
    },
    closeAddPopup() {
      this.newEventTitle = "";
      this.newEventTime = "";
      this.showAddPopup = false;
    },
    updateEvent(event) {
      const updateUrl = this.updateUrl + event.id;
      axios
          .put(updateUrl, event)
          .then(() => {
            const updatedEvent = this.events.find(e => e.id === event.id);
            if (updatedEvent) {
              updatedEvent.time = event.time;
            }
          })
          .catch((error) => {
            console.error('Error updating event:', error);
          });
    },
    checkScroll() {
      const timetable = document.querySelector('.timetable');
      this.shouldScroll = timetable.scrollHeight > 500;
    },
  },
  mounted() {
    axios
        .get(this.apiUrl)
        .then((response) => {
          this.events = response.data;
          this.checkScroll();
        })
        .catch((error) => {
          console.error('Error fetching events:', error);
        });
  },
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

.event-container {
  display: flex;
  gap: 10px;
  align-items: center;
  margin-left: 10px;
}

.event {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
  height: 42px;
  background-color: #555;
  padding: 5px;
  margin-bottom: 5px;
  border-radius: 12px;
  font-size: 14px;
  flex-grow: 1;
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
  display: flex;
  justify-content: center;
  align-items: center;
  color: white;
}

.time-slot {
  display: flex;
  align-items: center;
}

.time {
  padding: 20px;
  background-color: #333;
  margin-top: 10px;
  margin-bottom: 10px;
  border-radius: 12px;
  color: white;
  width: 100px;
  flex-shrink: 0;
  text-align: center;
}

.event-title {
  margin-bottom: 5px;
  width: 100%;
}

.timetable {
  max-height: 65vh;
  overflow-y: auto;
}

</style>

<style scoped>
::-webkit-scrollbar {
  width: 8px;
}

::-webkit-scrollbar-track {
  background-color: #242424;
}

::-webkit-scrollbar-thumb {
  background-color: #888;
  border-radius: 4px;
}

::-webkit-scrollbar-thumb:hover {
  background-color: #555;
}

/* Для Firefox */
::-moz-scrollbar {
  width: 8px;
}

::-moz-scrollbar-track {
  background-color: #242424;
}

::-moz-scrollbar-thumb {
  background-color: #888;
  border-radius: 4px;
}

::-moz-scrollbar-thumb:hover {
  background-color: #555;
}

/* Дополнительные стили для компонента */
.timetable.scrollable {
  overflow-y: scroll;
}

.timetable.scrollable::-webkit-scrollbar-track {
  background-color: #333;
}

.timetable.scrollable::-webkit-scrollbar-thumb {
  background-color: #666;
}

.timetable.scrollable::-webkit-scrollbar-thumb:hover {
  background-color: #888;
}

.timetable.scrollable::-moz-scrollbar-track {
  background-color: #333;
}

.timetable.scrollable::-moz-scrollbar-thumb {
  background-color: #666;
}

.timetable.scrollable::-moz-scrollbar-thumb:hover {
  background-color: #888;
}
</style>

