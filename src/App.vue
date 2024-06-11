<script setup>
import AgoraRTM from "agora-rtm-sdk";
import { v4 as uuidv4 } from "uuid";
import { ref, onMounted, nextTick, defineExpose } from "vue";

const APP_ID = "e4080ba51a8c4d17ae0f363d70dcc694";
const CHANNEL = "bilge-chat";
const client = AgoraRTM.createInstance(APP_ID);
const uid = uuidv4();
let channel;
const text = ref("");
const messages = ref([]);
const messagesRef = ref(null);

defineExpose({ messagesRef });

const appendMessage = async (message) => {
  messages.value.push(message);
  await nextTick();
  messagesRef.value.scrollTop = messagesRef.value.scrollHeight;
};

onMounted(async () => {
  await client.login({ uid, token: null });
  channel = await client.createChannel(CHANNEL);
  await channel.join();
  channel.on("ChannelMessage", ( message, senderId ) => {
    appendMessage({ text: message.text, uid: senderId });
  });
});

function sendMessage() {
  if (!text.value) return;
  channel.sendMessage({ text: text.value, type: "text" });
  appendMessage({ text: text.value, uid });
  text.value = "";
}
</script>

<template>
  <div class="panel">
    <div class="messages">
      <div class="inner">
        <div
          v-for="message in messages"
          :key="message.uid"
          class="message"
          ref="messagesRef"
        >
          <div v-if="message.uid === uid" class="user-self">You:</div>
          <div v-else class="user-them">Them:</div>
          <div class="text">{{ message.text }}</div>
        </div>
      </div>
    </div>
    <form @submit.prevent="sendMessage">
      <input v-model="text" />
      <button>+</button>
    </form>
  </div>
</template>

<style scoped>
body {
  margin: 0;
  height: 400px;
}

#app {
  padding: 2em;
  height: 100%;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  background: linear-gradient(
    90deg,
    rgba(188, 255, 147, 1) 0%,
    rgba(88, 245, 158, 1) 53%,
    rgba(0, 237, 69, 1) 100%
  );
}

.panel {
  display: flex;
  flex-direction: column;
  padding: 20px;
  margin: 0 auto;
  max-width: 300px;
  height: 300px;
  background: rgba(255, 255, 255, 0.7);
  box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.37);
  backdrop-filter: blur(4px);
  border-radius: 10px;
  border: 1px solid rgba(255, 255, 255, 0.18);
}
.messages {
  height: 100%;
  width: 100%;
  overflow-y: scroll;
  border-top-left-radius: 5px;
  border-top-right-radius: 5px;
  background-color: white;
}
.inner {
  padding: 10px;
}
.message {
  text-align: left;
  display: flex;
  margin-bottom: 6px;
}
.user-self {
  color: green;
}
.user-them {
  color: red;
}
form {
  position: relative;
  display: flex;
}
input {
  width: 100%;
  border: none;
  height: 20px;
  padding: 8px;
  border-top: 1px solid #999;
  border-radius: 0px;
  outline: none;
}
button {
  border: none;
  outline: none;
  background: none;
  position: absolute;
  right: 3px;
  top: 4px;
  font-size: 24px;
}
</style>
