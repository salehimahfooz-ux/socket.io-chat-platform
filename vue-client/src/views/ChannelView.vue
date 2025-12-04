<script setup>
import { onMounted, watch, ref, onUnmounted, computed } from "vue";
import { useRoute, useRouter } from "vue-router";
import { useMainStore } from "@/stores/main";
import Sidebar from "@/components/ChannelView/Sidebar.vue";
import ChatPanel from "@/components/ChannelView/ChatPanel.vue";

const store = useMainStore();
const route = useRoute();
const router = useRouter();
const isMobile = ref(window.innerWidth <= 768);

// تشخیص اینکه در صفحه چت هستیم یا لیست چت‌ها
const isChatRoute = computed(() => {
  return route.name === 'channel';
});

const channelId = computed(() => {
  return route.params.channelId;
});

function updateSelectChannel() {
  if (channelId.value) {
    return store.selectChannel(channelId.value);
  }
}

const handleResize = () => {
  isMobile.value = window.innerWidth <= 768;
};

onMounted(() => {
  window.addEventListener('resize', handleResize);
  updateSelectChannel();
});

onUnmounted(() => {
  window.removeEventListener('resize', handleResize);
});

watch(channelId, updateSelectChannel);
</script>

<template>
  <!-- حالت موبایل: اگر در صفحه چت هستیم (route نام 'channel') -->
  <div v-if="isMobile && isChatRoute" class="mobile-chat-view">
    <ChatPanel />
  </div>
  
  <!-- حالت موبایل: اگر در صفحه اصلی هستیم -->
  <div v-else-if="isMobile" class="mobile-sidebar-view">
    <Sidebar class="mobile-sidebar" />
  </div>

  <!-- حالت دسکتاپ: همیشه هر دو رو نشون بده -->
  <div v-else class="desktop-container">
    <Sidebar class="desktop-sidebar" />
    <div class="desktop-chat-area">
      <ChatPanel v-if="store.selectedChannel" />
      <div v-else class="desktop-empty-state">
        <div class="empty-content">
          <svg width="120" height="120" viewBox="0 0 24 24">
            <path fill="#25d366" d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-2 15l-5-5 1.41-1.41L10 14.17l7.59-7.59L19 8l-9 9z"/>
          </svg>
          <h3>Welcome to Chat</h3>
          <p>Select a conversation to start messaging</p>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* استایل‌ها بدون تغییر */
.desktop-container {
  display: flex;
  height: 100vh;
  background: #f0f2f5;
}

.desktop-sidebar {
  width: 400px;
  min-width: 350px;
  background: white;
  border-right: 1px solid #e0e0e0;
}

.desktop-chat-area {
  flex: 1;
  background: white;
}

.desktop-empty-state {
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #f8f9fa;
}

.empty-content {
  text-align: center;
  color: #667781;
  padding: 20px;
}

.empty-content h3 {
  margin: 20px 0 8px 0;
  color: #111b21;
  font-weight: 300;
}

/* موبایل */
.mobile-chat-view {
  height: 100vh;
  width: 100vw;
}

.mobile-sidebar-view {
  height: 100vh;
  width: 100vw;
}

.mobile-sidebar {
  height: 100%;
}

/* Responsive */
@media (max-width: 768px) {
  .desktop-container {
    display: none;
  }
}

@media (min-width: 769px) {
  .mobile-chat-view,
  .mobile-sidebar-view {
    display: none;
  }
}
</style>
