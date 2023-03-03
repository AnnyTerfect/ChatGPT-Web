<script setup>
import Chat from './components/chat.vue'
import { ref, onMounted, defineProps } from 'vue'

// Navigation bar
const showNav = ref(true)
const items = ref([
  { title: 'Change API Key' },
])

// About chat
const chatList = ref([
  { name: 'New Chat 1', id: 1, edit: false }
])
const activeChatId = ref(1)
const chatCounter = ref(1)

const props = defineProps({
  apiKey: {
    type: String,
    required: true
  }
})

function clickChat(id) {
  activeChatId.value = id
  chatList.value.forEach((chat) => chat.edit = false)
  chatList.value.filter((chat) => chat.id === id)[0].edit = true
}

function handleMenuClick(item, index) {
  if (index === 0) {
    const apiKey = prompt('Enter your API Key', localStorage.getItem('apiKey'))
    if (apiKey) {
      localStorage.setItem('apiKey', apiKey)
    }
    else {
      handleMenuClick(items[0], 0)
    }
  }
}

onMounted(() => {
  const apiKey = localStorage.getItem('apiKey')
  console.log(apiKey)
  if (!apiKey) {
    handleMenuClick(items[0], 0)
  }
})
</script>

<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
    >
      <v-app-bar-nav-icon @click="showNav = !showNav"></v-app-bar-nav-icon>
      <v-toolbar-title>Chat</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-menu>
        <template v-slot:activator="{ props }">
          <v-btn
            v-bind="props"
            icon
          >
            <v-icon>mdi-dots-vertical</v-icon>
          </v-btn>
        </template>
        <v-list>
          <v-list-item
            v-for="(item, index) in items"
            :key="index"
            :value="index"
            @click="handleMenuClick(item, index)"
          >
            <v-list-item-title>{{ item.title }}</v-list-item-title>
          </v-list-item>
        </v-list>
      </v-menu>
    </v-app-bar>

    <v-navigation-drawer
      app
      clipped
      width="350"
      max-width="100%"
      v-model="showNav"
    >
      <v-list>
        <div class="text-h6 ma-4">
          Chat List
        </div>

        <v-list-item
          v-for="chat in chatList"
          :key="chat.id"
          :value="chat.name"
          :active="activeChatId === chat.id"
          @click="clickChat(chat.id)"
          active-color="primary"
        >
          <template v-slot:append>
            <v-icon
              :icon="'mdi-delete'"
              @click="chatList.splice(i, 1)"
            ></v-icon>
          </template>

          <v-list-item-title v-if="!chat.edit" v-text="chat.name"></v-list-item-title>
          <v-text-field
            v-if="chat.edit"
            v-model="chatList.filter((chat) => chat.id === activeChatId)[0].name"
            label="Chat Name"
            single-line
            hide-details
            density="comfortable"
            @keyup.enter="chat.edit = false"
          />
        </v-list-item>

        <v-list-item
        >
          <v-btn
            style="width: 100%"
            color="primary"
            @click="chatList.push({ name: `New Chat ${++chatCounter}`, id: chatCounter })"
          >
            Add Chat <v-icon class="ml-1">mdi-plus</v-icon>
          </v-btn>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>

    <v-main>
      <div class="relative">
        <transition-group name="fade" tag="div" mode="out-in">
          <div
            v-for="chat in chatList"
            v-show="activeChatId === chat.id"
            :key="chat.id"
          >
            <Chat/>
          </div>
        </transition-group>
      </div>
    </v-main>
  </v-app>
</template>

<style scoped>
.fade-enter-active, .fade-leave-active {
  transition: opacity .5s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}
</style>