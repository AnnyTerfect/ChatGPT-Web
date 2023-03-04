<script setup>
import './style.css'
import Chat from './components/chat.vue'
import { ref, onMounted, computed } from 'vue'

// Navigation bar
const showNav = ref(false)
const items = ref([
  { title: 'Change API Key' },
])

// About chat
const chatList = ref([
  { name: 'New Chat 1', id: 1, edit: false }
])
const activeChatId = ref(1)
const chatCounter = ref(1)

function clickChat(id) {
  activeChatId.value = id
  chatList.value.forEach((chat) => chat.edit = false)
  chatList.value.filter((chat) => chat.id === id)[0].edit = true
}

// API key dialog
const apiKey = ref(null)
const dialog = ref(false)
const testing = ref(false)
const invalidKeyWarning = ref(false)
const testSuccessText = ref(false)
const errMsg = ref(null)

function saveAPIKey() {
  localStorage.setItem('apiKey', apiKey.value)
  dialog.value = false
}
function testAPIKey() {
  testSuccessText.value = false
  invalidKeyWarning.value = false
  testing.value = true
  errMsg.value = null

  fetch('https://api.openai.com/v1/engines', {
    method: 'get',
    headers: {
      'Content-Type': 'application/json',
      Authorization: 'Bearer ' + apiKey.value,
    }
  })
  .then((res) => {
    if (res.status) {
      if (res.status === 401) {
        invalidKeyWarning.value = true
        dialog.value = true
      }
      else if (res.status === 200) {
        testSuccessText.value = true
      }
    }
    else {
      throw res
    }
    testing.value = false
  })
  .catch((err) => {
    dialog.value = true
    testing.value = false
    errMsg.value = err
  })
}

onMounted(() => {
  apiKey.value = localStorage.getItem('apiKey')
  testAPIKey()
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
      <v-btn
        icon
        @click="dialog = true"
      >
        <v-icon>mdi-key</v-icon>
      </v-btn>
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
              @click="chatList.splice(chatList.indexOf(chat), 1)"
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

    <v-dialog
      v-model="dialog"
      persistent
      max-width="500px"
    >
      <template v-slot:activator="{ props }">
        <v-btn
          color="primary"
          v-bind="props"
        >
          Open Dialog
        </v-btn>
      </template>

      <v-card>
        <v-card-title>
          <span class="headline">Enter your API Key</span>
        </v-card-title>
        <v-card-text class="text-caption">
          You can find your API key at <a href="https://platform.openai.com/account/api-keys" target="_blank">Get API Key</a>
        </v-card-text>
        
        <v-card-text>
          <v-text-field
            v-model="apiKey"
            label="API Key"
            single-line
            hide-details
            density="comfortable"
            @input="testAPIKey"
          />
        </v-card-text>
        <v-card-text v-if="invalidKeyWarning" class="text-caption text-red">
          <v-icon
            class="mr-1"
            color="red"
          >
            mdi-alert-circle
          </v-icon>
          Invalid API Key
        </v-card-text>
        <v-card-text v-if="testSuccessText" class="text-caption text-green">
          <v-icon
            class="mr-1"
            color="green"
          >
            mdi-check-circle
          </v-icon>
          API Key is valid
        </v-card-text>
        <v-card-text v-if="errMsg !== null" class="text-caption text-red">
          <v-icon
            class="mr-1"
            color="red"
          >
            mdi-alert-circle
          </v-icon>
          {{ errMsg }}
        </v-card-text>
        <v-card-text v-if="testing" class="text-caption text-blue">
          <v-progress-circular
            :size="20"
            :width="2"
            color="white"
            indeterminate
          ></v-progress-circular>
          Testing API Key
        </v-card-text>
        <v-card-actions class="justify-center">
          <v-btn class="mx-3" color="" @click="testAPIKey">Test</v-btn>
          <v-btn class="mx-3" color="primary" :disabled="!testSuccessText" @click="saveAPIKey">Save</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
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