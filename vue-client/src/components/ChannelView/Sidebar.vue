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

const handleChannelClick = async (channelId) => {
  // اول channel رو select کن
  await store.selectChannel(channelId);
  
  // در موبایل: به صفحه چت برو
  const isMobile = window.innerWidth <= 768;
  if (isMobile) {
    // با استفاده از named route
    router.push({ 
      name: 'channel', 
      params: { channelId } 
    });
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
/* استایل‌ها همان قبلی */
.sidebar-container {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  background: white;
  padding: 16px;
}

/* بقیه استایل‌ها بدون تغییر... */
</style>
