<script setup>
import { computed, ref } from "vue";
import { useMainStore } from "@/stores/main";
import Username from "@/components/Username.vue";
import { socket } from "@/BackendService";
import VBtn from "@/components/common/VBtn.vue";
import PublicChannelLabel from "@/components/ChannelView/PublicChannelLabel.vue";
import PrivateChannelLabel from "@/components/ChannelView/PrivateChannelLabel.vue";

const store = useMainStore();
const isLoading = ref(false);
const isTyping = ref(false);

function submit() {
  const content = store.selectedChannel.messageInput;

  if (!content) {
    return;
  }

  store.sendMessage(content);

  store.selectedChannel.messageInput = "";

  onInput();
}

function onInput() {
  if (isTyping.value && store.selectedChannel.messageInput.length === 0) {
    isTyping.value = false;
    socket.emit("message:typing", {
      channelId: store.selectedChannelId,
      isTyping: false,
    });
  } else if (!isTyping.value && store.selectedChannel.messageInput.length > 0) {
    isTyping.value = true;
    socket.emit("message:typing", {
      channelId: store.selectedChannelId,
      isTyping: true,
    });
  }
}

async function loadMore() {
  isLoading.value = true;

  await store.loadMessagesForSelectedChannel("backward", true);

  setTimeout(() => {
    isLoading.value = false;
  }, 200);
}

function shouldPrintHeader(message, index) {
  return index === 0 || store.messages[index - 1].from !== message.from;
}

const someoneIsTyping = computed(() => {
  if (store.selectedChannel?.typingUsers.size) {
    const usernames = [];

    store.selectedChannel?.typingUsers.forEach(({ username }) => {
      usernames.push(username);
    });

    return usernames.join(", ") + " is typing";
  }
});
</script>

<template>
  <div
    v-if="store.isInitialized && store.selectedChannel"
    class="chat-panel-container"
  >
    <div class="channel-header">
      <PublicChannelLabel
        v-if="store.selectedChannel.type === 'public'"
        :channel="store.selectedChannel"
      ></PublicChannelLabel>

      <PrivateChannelLabel
        v-else
        :channel="store.selectedChannel"
      ></PrivateChannelLabel>
    </div>

    <div class="messages-container">
      <div class="load-more-btn" v-if="store.selectedChannel.hasMore">
        <v-btn
          color="primary"
          outlined
          :loading="isLoading"
          @click="loadMore"
          icon="arrow-up"
        >
          Load more
        </v-btn>
      </div>

      <div v-if="!store.messages.length" class="no-messages">No messages yet</div>

      <div class="messages-list">
        <div 
          v-for="(message, i) in store.messages" 
          :key="message.id"
          class="message-wrapper"
          :class="message.from === store.currentUser?.id ? 'sent' : 'received'"
        >
          <div v-if="shouldPrintHeader(message, i)" class="sender-header">
            <Username :id="message.from" class="sender-name" />
          </div>
          <div class="message-bubble">
            {{ message.content }}
          </div>
        </div>
      </div>
    </div>

    <div class="input-area">
      <div v-if="someoneIsTyping" class="typing-indicator">
        {{ someoneIsTyping }}
      </div>

      <form @submit.prevent="submit" class="message-form">
        <div class="input-wrapper">
          <textarea
            class="message-input"
            v-model="store.selectedChannel.messageInput"
            @keydown.exact.enter.prevent="submit"
            @input="onInput"
            rows="3"
            placeholder="Type a message..."
          ></textarea>

          <v-btn
            color="primary"
            class="submit-btn"
            type="submit"
            icon="send"
            :disabled="store.selectedChannel.messageInput === ''"
          >
          </v-btn>
        </div>
      </form>
    </div>
  </div>
</template>

<style scoped>
.chat-panel-container {
  height: 100%;
  display: flex;
  flex-direction: column;
  background: #efeae2;
}

.channel-header {
  background: #f0f2f5;
  padding: 16px;
  border-bottom: 1px solid #e0e0e0;
}

.messages-container {
  flex: 1;
  padding: 20px;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
}

.load-more-btn {
  text-align: center;
  margin-bottom: 20px;
}

.no-messages {
  text-align: center;
  color: #667781;
  padding: 40px 0;
}

.messages-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.message-wrapper {
  display: flex;
  flex-direction: column;
}

.message-wrapper.sent {
  align-items: flex-end;
}

.message-wrapper.received {
  align-items: flex-start;
}

.sender-header {
  margin-bottom: 4px;
  margin-left: 12px;
}

.sender-name {
  font-weight: 600;
  font-size: 13px;
  color: #667781;
}

/* حباب پیام واتساپ */
.message-bubble {
  max-width: 65%;
  padding: 8px 12px;
  border-radius: 7.5px;
  position: relative;
  box-shadow: 0 1px 0.5px rgba(0,0,0,0.13);
  word-wrap: break-word;
  font-size: 14.2px;
  line-height: 1.4;
}

/* پیام‌های خودت - سبز */
.message-wrapper.sent .message-bubble {
  background: #d9fdd3; /* سبز واتساپ */
  border-bottom-right-radius: 0;
  color: #111b21;
}

/* پیام‌های دیگران - سفید */
.message-wrapper.received .message-bubble {
  background: white; /* سفید واتساپ */
  border-top-left-radius: 0;
  color: #111b21;
}

/* دم حباب سبز (واتساپی) */
.message-wrapper.sent .message-bubble::after {
  content: '';
  position: absolute;
  right: -8px;
  bottom: 0;
  width: 0;
  height: 0;
  border-style: solid;
  border-width: 0 0 8px 8px;
  border-color: transparent transparent transparent #d9fdd3;
}

/* دم حباب سفید (واتساپی) */
.message-wrapper.received .message-bubble::after {
  content: '';
  position: absolute;
  left: -8px;
  top: 0;
  width: 0;
  height: 0;
  border-style: solid;
  border-width: 8px 8px 0 0;
  border-color: white transparent transparent transparent;
}

.input-area {
  background: #f0f2f5;
  padding: 16px;
  border-top: 1px solid #e0e0e0;
}

.typing-indicator {
  color: #25d366;
  font-size: 14px;
  margin-bottom: 8px;
  font-style: italic;
  padding: 0 8px;
}

.message-form {
  width: 100%;
}

.input-wrapper {
  position: relative;
  background: white;
  border-radius: 20px;
  padding: 8px 16px;
}

.message-input {
  width: 100%;
  border: none;
  outline: none;
  padding: 12px 8px;
  font-size: 15px;
  color: #111b21;
  background: transparent;
  resize: none;
}

.message-input::placeholder {
  color: #919191;
}

.submit-btn {
  position: absolute;
  right: 8px;
  bottom: 8px;
  background: #25d366;
  border: none;
  width: 36px;
  height: 36px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  color: white;
}

.submit-btn:disabled {
  background: #ccc;
  cursor: not-allowed;
}

/* برای موبایل */
@media (max-width: 768px) {
  .messages-container {
    padding: 10px;
  }
  
  .message-bubble {
    max-width: 80%;
  }
  
  .channel-header {
    padding: 12px;
  }
  
  .input-area {
    padding: 12px;
  }
}

/* اسکرول‌بار */
.messages-container::-webkit-scrollbar {
  width: 6px;
}

.messages-container::-webkit-scrollbar-track {
  background: transparent;
}

.messages-container::-webkit-scrollbar-thumb {
  background: #ccc;
  border-radius: 3px;
}

.messages-container::-webkit-scrollbar-thumb:hover {
  background: #aaa;
}
</style>
