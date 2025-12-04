<template>
  <!-- فقط قسمت messages را تغییر بده -->
  <div class="messages-container">
    <div 
      v-for="message in messages" 
      :key="message.id"
      class="message-wrapper"
      :class="message.sender_id === currentUser?.id ? 'sent' : 'received'"
    >
      <div class="message-bubble">
        {{ message.content }}
      </div>
    </div>
  </div>
</template>

<style scoped>
/* استایل واتساپ برای پیام‌ها */
.messages-container {
  padding: 20px;
  background: #efeae2;
}

.message-wrapper {
  margin-bottom: 10px;
  display: flex;
}

/* پیام فرستاده شده توسط خودت */
.message-wrapper.sent {
  justify-content: flex-end;
}

.message-wrapper.sent .message-bubble {
  background: #d9fdd3; /* سبز واتساپ */
  border-bottom-right-radius: 0;
}

/* پیام دریافتی از دیگران */
.message-wrapper.received {
  justify-content: flex-start;
}

.message-wrapper.received .message-bubble {
  background: white; /* سفید واتساپ */
  border-top-left-radius: 0;
}

/* حباب پیام */
.message-bubble {
  max-width: 65%;
  padding: 8px 12px;
  border-radius: 7.5px;
  position: relative;
  box-shadow: 0 1px 0.5px rgba(0,0,0,0.13);
  word-wrap: break-word;
}

/* دم حباب (واتساپی) */
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
</style>
