 <template>
  <div class="chat-app">
    <div class="chat-window">
      <div class="messages">
        <div
          v-for="(msg, index) in messages"
          :key="index"
          :class="['message', msg.sender === username ? 'sent' : 'received']"
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
import { defineComponent, ref, onMounted, onBeforeUnmount } from 'vue';

export default defineComponent({
  name: 'App',
  setup() {
    const messages = ref<{ sender: string; text: string; avatar: string }[]>([]);
    const newMessage = ref<string>('');
    const username = ref<string>(`User${Math.floor(Math.random() * 1000)}`);
    const avatars = [
      'https://via.placeholder.com/40/FF5733',
      'https://via.placeholder.com/40/33FF57',
      'https://via.placeholder.com/40/5733FF',
      'https://via.placeholder.com/40/Ff33A1',
      'https://via.placeholder.com/40/A1Ff33',
      'https://via.placeholder.com/40/33A1Ff',
      'https://via.placeholder.com/40/FFF333',
      'https://via.placeholder.com/40/33FFFf',
      'https://via.placeholder.com/40/5733A1',
    ];
    const userAvatar = ref<string>(
      avatars[Math.floor(Math.random() * avatars.length)]
    );

    let socket: WebSocket | null = null;

    const sendMessage = () => {
      if (socket && newMessage.value.trim()) {
        const message = {
          sender: username.value,
          text: newMessage.value,
          avatar: userAvatar.value,
        };

        socket.send(JSON.stringify(message));

        newMessage.value = '';
      }
    };

    onMounted(() => {
      const websocketUrl = import.meta.env.VITE_WEBSOCKET_URL;
      socket = new WebSocket(websocketUrl);

      socket.onmessage = (event: MessageEvent) => {
        const data = JSON.parse(event.data);

        const isDuplicate = messages.value.some(
          (msg) =>
            msg.text === data.text &&
            msg.sender === data.sender &&
            msg.avatar === data.avatar
        );

        if (!isDuplicate) {
          messages.value.push(data);
        }
      };

      socket.onclose = () => {
        console.log('WebSocket connection closed');
      };

      socket.onerror = (error: Event) => {
        console.error('WebSocket error:', error);
      };
    });

    onBeforeUnmount(() => {
      if (socket) {
        socket.close();
      }
    });

    return { messages, newMessage, sendMessage, username, avatars, userAvatar };
  },
});
</script>


<style>
.chat-app {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  font-family: Arial, sans-serif;
}

.chat-window {
  width: 400px;
  border: 1px solid #ccc;
  border-radius: 8px;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.messages {
  flex: 1;
  padding: 16px;
  overflow-y: auto;
  background: #f9f9f9;
  display: flex;
  flex-direction: column;
}

.message {
  display: flex;
  align-items: center;
  margin-bottom: 12px;
}

.message.sent {
  flex-direction: row-reverse;
  text-align: right;
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
}

.message-content {
  background: #e1f5fe;
  padding: 8px;
  border-radius: 8px;
  max-width: 70%;
}

.message.sent .message-content {
  background: #d1ffd6;
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
  background: #007bff;
  color: white;
  cursor: pointer;
}

button:hover {
  background: #0056b3;
}
</style>

