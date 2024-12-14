<template>
  <div class="chat-app">
    <div class="chat-window">
      <div class="messages">
        <div
          v-for="(msg, index) in messages"
          :key="index"
          :class="['message', msg.sender === username ? 'sent' : 'received', msg.animate ? 'animate' : '']"
        >
          <img
            :src="msg.avatar"
            alt="Avatar"
            class="avatar"
          />
          <div class="message-content">
            <p>{{ msg.text }}</p>
          </div>
        </div>
      </div>
      <div class="input-bar">
        <input
          v-model="newMessage"
          @keyup.enter="sendMessage"
          placeholder="Type a message..."
        />
        <button @click="sendMessage">Send</button>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';

export default defineComponent({
  name: 'App',
  data() {
    return {
      messages: [] as { sender: string; text: string; avatar: string, animate: boolean }[],
      newMessage: '',
      username: `User${Math.floor(Math.random() * 1000)}`,
      avatars: [
        'https://robohash.org/aaaaaa',
        'https://robohash.org/bbbbbb',
        'https://robohash.org/cccccc',
        'https://robohash.org/dddddd',
        'https://robohash.org/eeeeee',
        'https://robohash.org/ffffff',
        'https://robohash.org/gggggg',
        'https://robohash.org/hhhhhh',
        'https://robohash.org/iiiiii',
      ],
      userAvatar: '',
      socket: null as WebSocket | null,
    };
  },
  mounted() {
    this.userAvatar = this.avatars[Math.floor(Math.random() * this.avatars.length)];
    const websocketUrl = import.meta.env.VITE_WEBSOCKET_URL;
    this.socket = new WebSocket(websocketUrl);

    this.socket.onmessage = (event: MessageEvent) => {
      const data = JSON.parse(event.data);

      const isDuplicate = this.messages.some(
        (msg) =>
          msg.text === data.text &&
          msg.sender === data.sender &&
          msg.avatar === data.avatar
      );

      if (!isDuplicate) {
        this.messages.push(data);
      }
    };

    this.socket.onclose = () => {
      console.log('WebSocket connection closed');
    };

    this.socket.onerror = (error: Event) => {
      console.error('WebSocket error:', error);
    };
  },
  beforeUnmount() {
    if (this.socket) {
      this.socket.close();
    }
  },
  methods: {
    sendMessage() {
      if (this.socket && this.newMessage.trim()) {
        const message = {
          sender: this.username,
          text: this.newMessage,
          avatar: this.userAvatar,
        };

        this.socket.send(JSON.stringify(message));
        this.newMessage = '';
      }
    },
  },
});
</script>


<style>
.chat-app {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  font-family: Arial, sans-serif;
}

.chat-window {
  width: 700px;
  border: 1px solid #ccc;
  border-radius: 8px;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.messages {
  flex: 1;
  padding: 18px;
  overflow-y: auto;
  background: #f9f9f9;
  display: flex;
  flex-direction: column;
  opacity: 0;
  animation: slide-in 0.3s ease forwards;
}

@keyframes slide-in {
  0% {
    transform: translateY(20px);
    opacity: 0;
  }
  100% {
    transform: translateY(0);
    opacity: 1;
  }
}

.message {
  display: flex;
  align-items: flex-end;
  margin-bottom: 12px;
}



.message.sent {
  flex-direction: row-reverse;
  text-align: right;
  padding-top: 8px;
}

.message.received {
  flex-direction: row;
  text-align: left;
}

.avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  margin: 0 8px;
  border: 1px solid #171717;
}

.message-content {
  background: #e1f5fe;
  padding: 8px;
  border-radius: 8px;
  max-width: 70%;
}

.message-content > p {
  margin: 0;
}

.message.sent .message-content {
  background: #e5e5e5;
}

.input-bar {
  display: flex;
  border-top: 1px solid #ccc;
  padding: 8px;
}

input {
  flex: 1;
  padding: 8px;
  border: none;
  outline: none;
}

button {
  padding: 8px 16px;
  border: none;
  background: #171717;
  color: white;
  cursor: pointer;
}

button:hover {
  background: #000;
}


</style>

