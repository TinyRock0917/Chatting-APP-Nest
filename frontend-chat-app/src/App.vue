<script setup>
import { io } from 'socket.io-client'
import { onBeforeMount, ref } from 'vue';

const socket = io('http://localhost:3000');

const messages = ref([]);
const messageText = ref('');
const joined = ref(false);
const name = ref('');
const typingDisplay = ref('')

onBeforeMount(() => {
  socket.emit('findAllMessages', {}, (response) => {
    messages.value = response;
  });

  socket.on('message', (message) => {
    messages.value.push(message);
  })

  socket.on('typing', ({ name, isTyping }) => {
    if (isTyping) {
      typingDisplay.value = `${name} typing...`
    } else {
      typingDisplay.value = '';
    }
  })
})

const sendMessage = () => {
  socket.emit('createMessage', { text: messageText.value }, response => {
    console.log("response" + response);
    messageText.value = '';
  })
}

const join = () => {
  socket.emit('join', { name: name.value }, () => {
    joined.value = true;
  })
}

let timeout;
const emitTyping = () => {
  socket.emit('typing', { isTyping: true });
  timeout = setTimeout(() => {
    socket.emit('typing', { isTyping: false })
  }, 4000);
}
</script>

<template>
  <div class="container mx-auto w-3/6">
    <div class="flex justify-center text-white text-5xl p-4 border-b border-white mb-4">
      Chat App with WebSockets
    </div>
    <div class="flex justify-center text-white text-2xl p-4" v-if="!joined">
      <form @submit.prevent="join">
        <label for="name" class="mr-4">What's is your name?</label>
        <input type="text" v-model="name" class="mr-4 px-4 bg-transparent border border-green-600 py-2">
        <button type="submit"
          class="mr-4 bg-green-600 border border-green-600 hover:bg-green-900 hover:border-green-900 py-2 px-4 rounded">Submit</button>
      </form>
    </div>
    <div class="chat" v-else>
      <div class="flex justify-center bg-indigo-900 py-2 rounded-lg mt-2 text-white" v-if="messages.length === 0">
        No messages yet!
      </div>
      <div class="flex justify-center py-2" v-for="message in messages" v-else>
        <div class="text-2xl text-white text-center rounded-lg py-1 w-full"
          :class="message.name === name ? 'bg-green-600' : 'bg-indigo-500'">
          [{{ message.name }}]: {{ message.text }}
        </div>
      </div>

      <div v-if="typingDisplay" class="text-green-500 p-2 text-lg">
        {{ typingDisplay }}
      </div>
      <div class="border-b border-white mt-10"></div>


      <form @submit.prevent="sendMessage" class="flex justify-center mt-4">
        <input type="text" placeholder="Type a message..."
          class="bg-transparent border border-green-600 py-2 px-4 text-white focus:outline-none w-full"
          @input="emitTyping" v-model="messageText">
        <button type="submit"
          class="bg-green-600 border border-green-600 hover:bg-green-900 hover:border-green-900 py-2 px-4 text-white">
          Send
        </button>
      </form>

    </div>
  </div>
</template>

<style scoped>

</style>
