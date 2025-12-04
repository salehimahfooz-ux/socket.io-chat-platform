<script setup>
import { useMainStore } from "@/stores/main";
import BackendService from "@/BackendService";
import { useRouter } from "vue-router";
import VBtn from "@/components/common/VBtn.vue";
import PublicChannelLabel from "@/components/ChannelView/PublicChannelLabel.vue";
import PrivateChannelLabel from "@/components/ChannelView/PrivateChannelLabel.vue";

const store = useMainStore();
const router = useRouter();

async function logOut() {
  await BackendService.logOut();
  store.clear();
  await router.push({ name: "login" });
}

const handleChannelClick = (channelId) => {
  // اول channel رو select کن
  store.selectChannel(channelId);
  
  // در موبایل: به صفحه چت برو
  const isMobile = window.innerWidth <= 768;
  if (isMobile) {
    // یک تأخیر کوچک برای اینکه store به روز بشه
    setTimeout(() => {
      router.push({ name: 'channel', params: { channelId } });
    }, 50);
  }
}
</script>

<template>
  <div class="sidebar-container">
    <!-- Header -->
    <div class="sidebar-header">
      <img src="@/assets/logo.svg" class="logo" width="40" height="32" />
      <span class="app-name">Socket.IO chat</span>
    </div>

    <!-- User Profile (واتساپی) -->
    <div class="user-profile" v-if="store.currentUser">
      <img 
        :src="store.currentUser.avatar || `https://ui-avatars.com/api/?name=${store.currentUser.username}&background=random`" 
        class="user-avatar" 
        alt="User"
      />
      <div class="user-info">
        <span class="username">{{ store.currentUser.username }}</span>
        <span class="user-status">Online</span>
      </div>
    </div>

    <!-- Search Bar (واتساپی) -->
    <div class="search-bar">
      <div class="search-input-wrapper">
        <svg class="search-icon" width="20" height="20" viewBox="0 0 24 24">
          <path fill="#919191" d="M15.5 14h-.79l-.28-.27C15.41 12.59 16 11.11 16 9.5 16 5.91 13.09 3 9.5 3S3 5.91 3 9.5 5.91 16 9.5 16c1.61 0 3.09-.59 4.23-1.57l.27.28v.79l5 4.99L20.49 19l-4.99-5zm-6 0C7.01 14 5 11.99 5 9.5S7.01 5 9.5 5 14 7.01 14 9.5 11.99 14 9.5 14z"/>
        </svg>
        <input 
          type="text" 
          placeholder="Search or start new chat" 
          class="search-input"
        />
      </div>
    </div>

    <!-- CHANNELS Section -->
    <div class="section-header">
      <span>CHANNELS</span>
      <v-btn
        color="primary"
        class="add-btn"
        icon="plus"
        :disabled="!store.isInitialized"
        @click="store.showJoinOrCreateChannelModel = true"
        title="Add channel"
      >
      </v-btn>
    </div>

    <div class="channels-list">
      <div v-if="!store.isInitialized" v-for="_ in 4" class="channel-placeholder">
        <div class="placeholder-line"></div>
      </div>

      <div
        class="channel-item"
        v-for="channel in store.publicChannels"
        :key="channel.id"
      >
        <a
          class="channel-link"
          href="#"
          :class="store.isChannelSelected(channel.id) ? 'active' : ''"
          @click.prevent="handleChannelClick(channel.id)"
        >
          <PublicChannelLabel :channel="channel" class="channel-label" />
          
          <span class="unread-badge" v-if="channel.unreadCount > 0">
            {{ channel.unreadCount }}
          </span>
        </a>
      </div>
    </div>

    <!-- DIRECT MESSAGES Section -->
    <div class="section-header">
      <span>DIRECT MESSAGES</span>
      <v-btn
        color="primary"
        class="add-btn"
        icon="plus"
        @click="store.showSearchUserModal = true"
        :disabled="!store.isInitialized"
        title="Add user"
      >
      </v-btn>
    </div>

    <div class="channels-list">
      <div v-if="!store.isInitialized" v-for="_ in 4" class="channel-placeholder">
        <div class="placeholder-line"></div>
      </div>

      <div
        class="channel-item"
        v-for="channel in store.privateChannels"
        :key="channel.id"
      >
        <a
          class="channel-link"
          href="#"
          :class="store.isChannelSelected(channel.id) ? 'active' : ''"
          @click.prevent="handleChannelClick(channel.id)"
        >
          <PrivateChannelLabel :channel="channel" class="channel-label" />
          
          <span class="unread-badge" v-if="channel.unreadCount > 0">
            {{ channel.unreadCount }}
          </span>
        </a>
      </div>
    </div>

    <!-- Logout Button -->
    <div class="logout-section">
      <button class="logout-btn" @click="logOut" :disabled="!store.isInitialized">
        <svg width="20" height="20" viewBox="0 0 24 24" class="logout-icon">
          <path fill="currentColor" d="M16 13v-2H7V8l-5 4 5 4v-3z"/>
          <path fill="currentColor" d="M20 3h-9c-1.103 0-2 .897-2 2v4h2V5h9v14h-9v-4H9v4c0 1.103.897 2 2 2h9c1.103 0 2-.897 2-2V5c0-1.103-.897-2-2-2z"/>
        </svg>
        <span>Log out</span>
      </button>
    </div>
  </div>
</template>

<style scoped>
.sidebar-container {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  background: white;
  padding: 16px;
}

/* Header */
.sidebar-header {
  display: flex;
  align-items: center;
  gap: 12px;
  padding-bottom: 16px;
  border-bottom: 1px solid #f0f0f0;
  margin-bottom: 16px;
}

.logo {
  flex-shrink: 0;
}

.app-name {
  font-size: 18px;
  font-weight: 600;
  color: #111b21;
}

/* User Profile */
.user-profile {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px;
  background: #f8f9fa;
  border-radius: 8px;
  margin-bottom: 16px;
}

.user-avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  object-fit: cover;
}

.user-info {
  display: flex;
  flex-direction: column;
}

.username {
  font-weight: 600;
  color: #111b21;
  font-size: 14px;
}

.user-status {
  font-size: 12px;
  color: #25d366;
}

/* Search Bar */
.search-bar {
  margin-bottom: 20px;
}

.search-input-wrapper {
  position: relative;
  background: #f0f2f5;
  border-radius: 8px;
  padding: 10px 16px;
}

.search-icon {
  position: absolute;
  left: 16px;
  top: 50%;
  transform: translateY(-50%);
}

.search-input {
  width: 100%;
  border: none;
  outline: none;
  background: transparent;
  padding-left: 28px;
  font-size: 14px;
  color: #111b21;
}

.search-input::placeholder {
  color: #919191;
}

/* Sections */
.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
  padding: 0 4px;
}

.section-header span {
  font-size: 14px;
  font-weight: 600;
  color: #667781;
  text-transform: uppercase;
}

.add-btn {
  background: #25d366;
  border: none;
  width: 28px;
  height: 28px;
  border-radius: 50%;
  padding: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  color: white;
  min-width: auto;
}

.add-btn:disabled {
  background: #ccc;
  cursor: not-allowed;
}

/* Channels List */
.channels-list {
  flex: 1;
  overflow-y: auto;
  margin-bottom: 20px;
}

.channel-item {
  margin-bottom: 4px;
}

.channel-link {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10px 12px;
  border-radius: 8px;
  text-decoration: none;
  color: #111b21;
  transition: background 0.2s;
  cursor: pointer;
}

.channel-link:hover {
  background: #f5f5f5;
}

.channel-link.active {
  background: #e8f4f9;
}

.channel-label {
  flex: 1;
}

.unread-badge {
  background: #25d366;
  color: white;
  font-size: 12px;
  font-weight: 600;
  padding: 2px 6px;
  border-radius: 10px;
  min-width: 18px;
  text-align: center;
}

/* Placeholder */
.channel-placeholder {
  padding: 10px 12px;
}

.placeholder-line {
  height: 20px;
  background: linear-gradient(90deg, #f0f0f0 25%, #e0e0e0 50%, #f0f0f0 75%);
  background-size: 200% 100%;
  animation: loading 1.5s infinite;
  border-radius: 4px;
}

@keyframes loading {
  0% {
    background-position: 200% 0;
  }
  100% {
    background-position: -200% 0;
  }
}

/* Logout */
.logout-section {
  padding-top: 16px;
  border-top: 1px solid #f0f0f0;
  margin-top: auto;
}

.logout-btn {
  display: flex;
  align-items: center;
  gap: 10px;
  width: 100%;
  padding: 12px;
  background: #f8f9fa;
  border: none;
  border-radius: 8px;
  color: #111b21;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  transition: background 0.2s;
}

.logout-btn:hover:not(:disabled) {
  background: #e9ecef;
}

.logout-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.logout-icon {
  color: #667781;
}

/* Scrollbar */
.channels-list::-webkit-scrollbar {
  width: 4px;
}

.channels-list::-webkit-scrollbar-track {
  background: transparent;
}

.channels-list::-webkit-scrollbar-thumb {
  background: #ccc;
  border-radius: 2px;
}

.channels-list::-webkit-scrollbar-thumb:hover {
  background: #aaa;
}
</style>
