<script>
import { ref } from "vue";
import io from "socket.io-client";
const socket = io("http://localhost:5173");

socket.on("connect_error", (data) => {
  console.error(data);
});

export default {
  name: "Homeview",
  components: "HelloWorld",
  data() {
    return {
      messages: [],
      author: null,
      message: null,
      rooms: [],
      room_id: null,
      creating_room: null,
    };
  },
  methods: {
    createRoom() {
      socket.emit("createroom", {
        name: this.creating_room,
      });
      this.creating_room = null;
    },
    joinRoom(room_id) {
      socket.emit("leave_room", { room_id: this.room_id });

      this.room_id = room_id;
      socket.emit("join_room", { room_id: this.room_id });
    },
    sendMessage() {
      const message = {
        name: this.author,
        message: this.message,
        room_id: this.room_id,
      };
      socket.emit("message", message);
      this.message = null;
    },
    // send() {
    //   socket.emit("message", {
    //     message: this.message,
    //     room_id: this.room_id,
    //   });
    // },
  },
  mounted() {
    socket.on("connect_error", (error) => {
      console.error(error);
    });

    socket.on("history", (data) => {
      this.messages = data;
    });

    socket.on("get-rooms", (data) => {
      this.rooms = data;
    });

    socket.on("rooms_list_changed", (data) => {
      this.rooms.push(data);
    });

    socket.on("message", (data) => {
      this.messages.unshift(data);
    });
  },
};
</script>

<template>
  <div class="home">
    <div class="wrapper">
      <div class="chat">
        <p>Чат</p>
        <div class="chat-history">
          <div class="chat-history-item" v-for="message in messages">
            <h3>{{ message.name }}</h3>
            <p>{{ message.message }}</p>
          </div>
        </div>
        <p>Сообщение</p>
        <div style="text-align: left">
          <input
            type="text"
            placeholder="Your Name"
            style="margin-left: 5px"
            v-model="author"
          />
        </div>
        <div>
          <input type="text" style="width: 250px" v-model="message" />
          <button @click="sendMessage">Отправить</button>
        </div>
      </div>
      <div class="rooms">
        <p>Комнаты</p>
        <div class="rooms-list">
          <div class="rooms-list-item" v-for="room in rooms">
            <p
              @click="$event => joinRoom(room.id)"
              :style="room.id === room_id ? 'text-decoration: underline' : ''"
            >
              {{ room.name }}
            </p>
          </div>
        </div>
        <p>Создать комнату</p>
        <div>
          <input type="text" style="width: 250px" v-model="creating_room" />
          <button @click="createRoom">Создать</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.wrapper {
  width: 800px;
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  border: 1px solid gray;
}

.chat-history,
.rooms-list {
  width: 350px;
  height: 300px;
  background-color: #f5f5f5;
  overflow-y: scroll;
}

.rooms-list-item > p {
  cursor: pointer;
}

.chat-history-item {
  padding: 10px;
  border: 1px solid #bebebe;
  text-align: left;
}
</style>
