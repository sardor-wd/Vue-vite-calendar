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
            <div class="day" v-for="day in days" :key="day" @drop="dropEvent(day, $event)" @dragover.prevent>
                <div class="day-number" :class="{ 'current-day': isCurrentDay(day) }">{{ day }}</div>
                <div class="events">
                    <div class="event" v-for="event in getEventsByDay(day)" :key="event.id" draggable="true"
                         @dragstart="dragEvent(event.id)">
                        {{ event.title }}
                        <button @click="openDeletePopup(event.id)" class="delete-button">Delete</button>
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

        <div v-if="showDeletePopup" class="popup">
            <div class="popup-content">
                <h3>Delete Event</h3>
                <p>Are you sure you want to delete this event?</p>
                <div class="popup-buttons">
                    <button @click="deleteEvent" class="confirm-button">Delete</button>
                    <button @click="closeDeletePopup" class="cancel-button">Cancel</button>
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
            newEventTitle: '',
            newEventDate: '',
            showAddPopup: false,
            showDeletePopup: false,
            selectedEventId: null,
        };
    },
    computed: {
        currentMonth() {
            return this.currentDate.toLocaleString('default', {month: 'long', year: 'numeric'});
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
        openAddPopup() {
            this.showAddPopup = true;
        },
        closeAddPopup() {
            this.showAddPopup = false;
            this.clearEventFields();
        },
        openDeletePopup(eventId) {
            this.showDeletePopup = true;
            this.selectedEventId = eventId;
        },
        closeDeletePopup() {
            this.showDeletePopup = false;
            this.selectedEventId = null;
        },
        addEvent() {
            if (this.newEventTitle && this.newEventDate) {
                const newEvent = {
                    id: Math.random().toString(36).substr(2, 9), // Generate a random ID
                    title: this.newEventTitle,
                    date: new Date(this.newEventDate),
                };
                this.events.push(newEvent);
                this.clearEventFields();
                this.closeAddPopup();
            }
        },
        deleteEvent() {
            this.events = this.events.filter((event) => event.id !== this.selectedEventId);
            this.closeDeletePopup();
        },
        clearEventFields() {
            this.newEventTitle = '';
            this.newEventDate = '';
        },
        dragEvent(eventId) {
            event.dataTransfer.setData('text/plain', eventId);
        },
        dropEvent(day, event) {
            const eventId = event.dataTransfer.getData('text/plain');
            const foundEvent = this.events.find((e) => e.id === eventId);

            if (foundEvent) {
                foundEvent.date = new Date(this.currentDate.getFullYear(), this.currentDate.getMonth(), day);
            }
        },
    },
};
</script>

<style scoped>
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
    background-color: #00bfa5;
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
</style>
