<!-- vue-client/src/views/ChannelView.vue -->
<template>
  <div class="whatsapp-container" :class="{ 'mobile-view': isMobile }">
    
    <!-- Desktop Layout -->
    <div v-if="!isMobile" class="desktop-layout">
      <!-- Sidebar (لیست چت‌ها) -->
      <div class="sidebar">
        <!-- استفاده از کامپوننت Sidebar موجود -->
        <div class="whatsapp-sidebar">
          <div class="sidebar-header">
            <div class="user-profile">
              <img :src="userAvatar" class="user-avatar" alt="User">
              <span class="username">{{ currentUser?.username || 'User' }}</span>
            </div>
            <button @click="showNewChannel = true" class="new-chat-btn" title="New chat">
              <svg width="24" height="24" viewBox="0 0 24 24">
                <path fill="currentColor" d="M19 13h-6v6h-2v-6H5v-2h6V5h2v6h6v2z"/>
              </svg>
            </button>
          </div>
          
          <!-- Chat List -->
          <div class="chat-list">
            <div 
              v-for="channel in channels" 
              :key="channel.id"
              class="chat-item"
              :class="{ active: selectedChannel?.id === channel.id }"
              @click="handleSelectChannel(channel)"
            >
              <div class="chat-avatar">
                <div class="channel-avatar">#</div>
              </div>
              
              <div class="chat-info">
                <div class="chat-header">
                  <h4 class="chat-name">#{{ channel.name }}</h4>
                  <span class="chat-time">{{ formatTime(channel.last_message_at) }}</span>
                </div>
                <p class="last-message">{{ getLastMessage(channel) }}</p>
              </div>
            </div>
          </div>
        </div>
      </div>
      
      <!-- Main Chat Area -->
      <div class="main-chat">
        <div v-if="selectedChannel" class="whatsapp-chat">
          <!-- Chat Header -->
          <div class="chat-header">
            <div class="header-content">
              <div class="chat-avatar">
                <div class="channel-avatar">#</div>
              </div>
              <div class="chat-info">
                <h2>#{{ selectedChannel.name }}</h2>
                <p class="member-count">{{ selectedChannel.member_count || 1 }} members</p>
              </div>
            </div>
          </div>

          <!-- Messages Container -->
          <div class="messages-container" ref="messagesContainer">
            <!-- Messages -->
            <div 
              v-for="message in currentMessages" 
              :key="message.id" 
              class="message-wrapper"
              :class="{
                'sent': message.sender_id === currentUser?.id,
                'received': message.sender_id !== currentUser?.id
              }"
            >
              <!-- Message Bubble -->
              <div class="message-bubble">
                <div class="message-content">
                  <p class="message-text">{{ message.content }}</p>
                  <div class="message-footer">
                    <span class="message-time">{{ formatMessageTime(message.created_at) }}</span>
                    <span v-if="message.sender_id === currentUser?.id" class="message-status">
                      ✓✓
                    </span>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Message Input Area -->
          <div class="message-input-area">
            <div class="input-wrapper">
              <input
                v-model="newMessage"
                @keyup.enter="handleSendMessage"
                type="text"
                placeholder="Type a message..."
                class="message-input"
                ref="messageInput"
              />
              <button @click="handleSendMessage" class="send-btn">
                <svg width="24" height="24" viewBox="0 0 24 24">
                  <path fill="currentColor" d="M2.01 21L23 12 2.01 3 2 10l15 2-15 2z"/>
                </svg>
              </button>
            </div>
          </div>
        </div>
        
        <!-- Empty State -->
        <div v-else class="empty-state">
          <div class="empty-content">
            <div class="empty-icon">
              <svg width="120" height="120" viewBox="0 0 24 24">
                <path fill="#25d366" d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-2 15l-5-5 1.41-1.41L10 14.17l7.59-7.59L19 8l-9 9z"/>
              </svg>
            </div>
            <h3>Welcome to WhatsApp-like Chat</h3>
            <p>Select a conversation to start messaging</p>
          </div>
        </div>
      </div>
    </div>

    <!-- Mobile Layout -->
    <div v-else class="mobile-layout">
      <div v-if="!selectedChannel" class="mobile-sidebar">
        <!-- Mobile Sidebar -->
        <div class="mobile-sidebar-content">
          <div class="mobile-header">
            <h2>Chats</h2>
            <button @click="showNewChannel = true" class="new-chat-btn">
              <svg width="24" height="24" viewBox="0 0 24 24">
                <path fill="currentColor" d="M19 13h-6v6h-2v-6H5v-2h6V5h2v6h6v2z"/>
              </svg>
            </button>
          </div>
          
          <div class="mobile-chat-list">
            <div 
              v-for="channel in channels" 
              :key="channel.id"
              class="chat-item"
              @click="handleSelectChannel(channel)"
            >
              <div class="chat-avatar">
                <div class="channel-avatar">#</div>
              </div>
              
              <div class="chat-info">
                <div class="chat-header">
                  <h4 class="chat-name">#{{ channel.name }}</h4>
                  <span class="chat-time">{{ formatTime(channel.last_message_at) }}</span>
                </div>
                <p class="last-message">{{ getLastMessage(channel) }}</p>
              </div>
            </div>
          </div>
        </div>
      </div>
      
      <div v-else class="mobile-chat">
        <!-- Mobile Chat Header -->
        <div class="mobile-chat-header">
          <button @click="selectedChannel = null" class="back-button">
            ←
          </button>
          <div class="header-content">
            <div class="chat-avatar">
              <div class="channel-avatar">#</div>
            </div>
            <div class="chat-info">
              <h2>#{{ selectedChannel.name }}</h2>
              <p class="member-count">{{ selectedChannel.member_count || 1 }} members</p>
            </div>
          </div>
        </div>

        <!-- Mobile Messages -->
        <div class="mobile-messages-container" ref="mobileMessagesContainer">
          <div 
            v-for="message in currentMessages" 
            :key="message.id" 
            class="message-wrapper"
            :class="{
              'sent': message.sender_id === currentUser?.id,
              'received': message.sender_id !== currentUser?.id
            }"
          >
            <div class="message-bubble">
              <div class="message-content">
                <p class="message-text">{{ message.content }}</p>
                <div class="message-footer">
                  <span class="message-time">{{ formatMessageTime(message.created_at) }}</span>
                  <span v-if="message.sender_id === currentUser?.id" class="message-status">
                    ✓✓
                  </span>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- Mobile Message Input -->
        <div class="mobile-message-input">
          <input
            v-model="newMessage"
            @keyup.enter="handleSendMessage"
            type="text"
            placeholder="Type a message..."
            class="message-input"
          />
          <button @click="handleSendMessage" class="send-btn">
            <svg width="24" height="24" viewBox="0 0 24 24">
              <path fill="currentColor" d="M2.01 21L23 12 2.01 3 2 10l15 2-15 2z"/>
            </svg>
          </button>
        </div>
      </div>
    </div>

    <!-- Modals - استفاده از کامپوننت‌های موجود -->
    <JoinOrCreateChannelModal 
      v-if="showNewChannel"
      @close="showNewChannel = false"
      @join="handleJoinChannel"
    />
    
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted, nextTick } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { storeToRefs } from 'pinia'
import { useMainStore } from '@/stores/main.js'

// استفاده از کامپوننت‌های موجود در پروژه
import JoinOrCreateChannelModal from '@/components/ChannelView/JoinOrCreateChannelModal.vue'

const route = useRoute()
const router = useRouter()
const mainStore = useMainStore()

// State
const selectedChannel = ref(null)
const newMessage = ref('')
const isMobile = ref(window.innerWidth <= 768)
const showNewChannel = ref(false)
const messagesContainer = ref(null)
const mobileMessagesContainer = ref(null)

// از store استفاده کنیم
const { channels, currentUser } = storeToRefs(mainStore)

// Computed
const currentMessages = computed(() => {
  if (!selectedChannel.value) return []
  return mainStore.messages[selectedChannel.value.id] || []
})

const userAvatar = computed(() => {
  return currentUser.value?.avatar || `https://ui-avatars.com/api/?name=${currentUser.value?.username || 'User'}&background=random`
})

// Methods
const handleSelectChannel = (channel) => {
  selectedChannel.value = channel
  // Load messages
  mainStore.loadMessages(channel.id)
  
  // Update URL برای دسکتاپ
  if (!isMobile.value) {
    router.push({ name: 'channel', params: { channelId: channel.id } })
  }
  
  // Scroll to bottom
  nextTick(() => {
    const container = isMobile.value ? mobileMessagesContainer.value : messagesContainer.value
    if (container) {
      container.scrollTop = container.scrollHeight
    }
  })
}

const handleSendMessage = () => {
  const content = newMessage.value.trim()
  if (!content || !selectedChannel.value) return
  
  mainStore.sendMessage(selectedChannel.value.id, content)
  newMessage.value = ''
  
  // Scroll to bottom
  nextTick(() => {
    const container = isMobile.value ? mobileMessagesContainer.value : messagesContainer.value
    if (container) {
      container.scrollTop = container.scrollHeight
    }
  })
}

const handleJoinChannel = (channel) => {
  showNewChannel.value = false
  // Add channel to store
  mainStore.joinChannel(channel.id)
  // Select the new channel
  handleSelectChannel(channel)
}

const getLastMessage = (channel) => {
  if (!channel.last_message) {
    return 'No messages yet'
  }
  
  // Truncate long messages
  if (channel.last_message.length > 35) {
    return channel.last_message.substring(0, 35) + '...'
  }
  
  return channel.last_message
}

const formatTime = (timestamp) => {
  if (!timestamp) return ''
  
  const date = new Date(timestamp)
  const now = new Date()
  const diff = now - date
  
  // Today
  if (diff < 24 * 60 * 60 * 1000) {
    return date.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })
  }
  
  // This week
  if (diff < 7 * 24 * 60 * 60 * 1000) {
    const days = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat']
    return days[date.getDay()]
  }
  
  // Older
  return date.toLocaleDateString([], { month: 'short', day: 'numeric' })
}

const formatMessageTime = (timestamp) => {
  if (!timestamp) return ''
  const date = new Date(timestamp)
  return date.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })
}

// Handle resize
const handleResize = () => {
  isMobile.value = window.innerWidth <= 768
}

onMounted(() => {
  window.addEventListener('resize', handleResize)
  
  // Load channels
  mainStore.loadChannels()
  
  // Check URL for channel
  if (route.params.channelId) {
    const channel = channels.value.find(c => c.id === parseInt(route.params.channelId))
    if (channel) {
      selectedChannel.value = channel
      mainStore.loadMessages(channel.id)
    }
  }
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
})
</script>

<style scoped>
/* استایل‌های WhatsApp */
.whatsapp-container {
  height: 100vh;
  background: #f0f2f5;
}

/* Desktop Layout */
.desktop-layout {
  display: flex;
  height: 100%;
  max-width: 1600px;
  margin: 0 auto;
}

.sidebar {
  width: 400px;
  min-width: 350px;
  background: white;
  border-right: 1px solid #e0e0e0;
  display: flex;
  flex-direction: column;
}

.sidebar-header {
  padding: 16px;
  background: #f8f9fa;
  border-bottom: 1px solid #e0e0e0;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.user-profile {
  display: flex;
  align-items: center;
  gap: 12px;
}

.user-avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  object-fit: cover;
}

.username {
  font-weight: 600;
  color: #333;
}

.new-chat-btn {
  background: #25d366;
  border: none;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  color: white;
  transition: background 0.2s;
}

.new-chat-btn:hover {
  background: #128c7e;
}

.chat-list {
  flex: 1;
  overflow-y: auto;
}

.chat-item {
  display: flex;
  padding: 12px 16px;
  border-bottom: 1px solid #f5f5f5;
  cursor: pointer;
  transition: background 0.2s;
}

.chat-item:hover {
  background: #f8f9fa;
}

.chat-item.active {
  background: #e8f4f9;
}

.chat-avatar {
  margin-right: 12px;
}

.channel-avatar {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  background: #25d366;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 20px;
  font-weight: bold;
}

.chat-info {
  flex: 1;
  min-width: 0;
}

.chat-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 4px;
}

.chat-name {
  margin: 0;
  font-size: 16px;
  font-weight: 500;
  color: #111b21;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.chat-time {
  font-size: 12px;
  color: #667781;
}

.last-message {
  margin: 0;
  font-size: 14px;
  color: #667781;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* Main Chat Area */
.main-chat {
  flex: 1;
  background: white;
  display: flex;
  flex-direction: column;
}

.whatsapp-chat {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.chat-header {
  background: #f0f2f5;
  padding: 16px;
  border-bottom: 1px solid #e0e0e0;
}

.header-content {
  display: flex;
  align-items: center;
  gap: 12px;
}

.chat-info h2 {
  margin: 0;
  font-size: 16px;
  font-weight: 600;
  color: #111b21;
}

.member-count {
  margin: 2px 0 0 0;
  font-size: 13px;
  color: #667781;
}

.messages-container {
  flex: 1;
  padding: 20px;
  overflow-y: auto;
  background: #efeae2;
  background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="100" height="100" opacity="0.05"><path d="M50,0 L100,50 L50,100 L0,50 Z" fill="%23000"/></svg>');
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.message-wrapper {
  display: flex;
  margin-bottom: 4px;
}

.message-wrapper.sent {
  justify-content: flex-end;
}

.message-wrapper.received {
  justify-content: flex-start;
}

.message-bubble {
  max-width: 65%;
}

.message-content {
  padding: 8px 12px;
  border-radius: 7.5px;
  background: white;
  box-shadow: 0 1px 0.5px rgba(0,0,0,0.13);
}

.message-wrapper.sent .message-content {
  background: #d9fdd3;
  border-bottom-right-radius: 0;
}

.message-wrapper.received .message-content {
  background: white;
  border-top-left-radius: 0;
}

.message-text {
  margin: 0 0 2px 0;
  font-size: 14.2px;
  line-height: 19px;
  color: #111b21;
}

.message-footer {
  display: flex;
  justify-content: flex-end;
  align-items: center;
}

.message-time {
  font-size: 11px;
  color: #667781;
  margin-right: 4px;
}

.message-status {
  font-size: 11px;
  color: #4fc3f7;
}

.message-input-area {
  background: #f0f2f5;
  padding: 12px 16px;
  border-top: 1px solid #e0e0e0;
}

.input-wrapper {
  display: flex;
  align-items: center;
  gap: 12px;
  background: white;
  border-radius: 20px;
  padding: 5px 15px;
}

.message-input {
  flex: 1;
  border: none;
  outline: none;
  padding: 12px 8px;
  font-size: 15px;
  color: #111b21;
  background: transparent;
}

.send-btn {
  background: none;
  border: none;
  cursor: pointer;
  padding: 8px;
  color: #25d366;
  display: flex;
  align-items: center;
  justify-content: center;
}

.empty-state {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #f8f9fa;
}

.empty-content {
  text-align: center;
  color: #667781;
}

.empty-content h3 {
  margin: 20px 0 8px 0;
  color: #111b21;
}

/* Mobile Layout */
.mobile-layout {
  height: 100vh;
  width: 100vw;
}

.mobile-sidebar {
  height: 100%;
}

.mobile-header {
  padding: 16px;
  background: #f8f9fa;
  border-bottom: 1px solid #e0e0e0;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.mobile-header h2 {
  margin: 0;
  font-size: 20px;
  color: #111b21;
}

.mobile-chat-list {
  flex: 1;
  overflow-y: auto;
}

.mobile-chat {
  height: 100%;
  display: flex;
  flex-direction: column;
}

.mobile-chat-header {
  background: #f0f2f5;
  padding: 12px 16px;
  border-bottom: 1px solid #e0e0e0;
  display: flex;
  align-items: center;
  gap: 12px;
}

.back-button {
  background: none;
  border: none;
  cursor: pointer;
  padding: 8px;
  color: #54656f;
  font-size: 20px;
}

.mobile-messages-container {
  flex: 1;
  padding: 16px;
  overflow-y: auto;
  background: #efeae2;
  background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="100" height="100" opacity="0.05"><path d="M50,0 L100,50 L50,100 L0,50 Z" fill="%23000"/></svg>');
}

.mobile-message-input {
  background: #f0f2f5;
  padding: 12px 16px;
  border-top: 1px solid #e0e0e0;
  display: flex;
  gap: 12px;
}

/* Responsive */
@media (max-width: 768px) {
  .desktop-layout {
    display: none;
  }
  
  .chat-item {
    padding: 10px 12px;
  }
  
  .channel-avatar {
    width: 40px;
    height: 40px;
    font-size: 18px;
  }
  
  .message-bubble {
    max-width: 80%;
  }
}

@media (min-width: 769px) {
  .mobile-layout {
    display: none;
  }
}

/* Scrollbar styling */
::-webkit-scrollbar {
  width: 6px;
}

::-webkit-scrollbar-track {
  background: #f1f1f1;
}

::-webkit-scrollbar-thumb {
  background: #c1c1c1;
  border-radius: 3px;
}

::-webkit-scrollbar-thumb:hover {
  background: #a8a8a8;
}
</style>
