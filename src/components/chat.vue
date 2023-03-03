<script setup>
import { ref } from 'vue'
import { marked } from 'marked';

const content = ref('')
const chatList = ref([])
const chatListElems = ref(null)
const sending = ref(false)

function sendcontent() {
  const apiKey = localStorage.getItem('apiKey')
  if (content.value === '') return
  sending.value = true
  chatList.value.push({role: 'user', content: content.value })
  content.value = ''
  setTimeout(() => {
    chatListElems.value[chatListElems.value.length - 1].scrollIntoView({ behavior: 'smooth' })
  }, 0)

  fetch('https://api.openai.com/v1/chat/completions', {
    method: 'post',
    headers: {
      'Content-Type': 'application/json',
      Authorization: 'Bearer ' + apiKey,
    },
    body: JSON.stringify({
      model: 'gpt-3.5-turbo',
      messages: chatList.value.filter((chat) => chat.role == 'user' || chat.role == 'assistant')
    })
  })
  .then((res) => {
    return res.json()
  })
  .then((res) => {
    if (res.error) throw `${res.error.code}: ${res.error.message}`
    sending.value = false
    chatList.value.push({role: 'assistant', content: res.choices[0].message.content })
    chatListElems.value[chatListElems.value.length - 1].scrollIntoView({ behavior: 'smooth' })
    setTimeout(() => {
      chatListElems.value[chatListElems.value.length - 1].scrollIntoView({ behavior: 'smooth' })
    }, 300)
  })
  .catch((err) => {
    console.log(err)
    chatList.value.push({role: 'error', content: err })
    sending.value = false
  })
}
</script>

<template>
  <v-row style="max-width: 900px; margin: auto">
    <v-col cols="12">
      <v-card flat>
        <div>
          <v-list>
            <div
              v-if="chatList.length === 0"
              class="text-center"
            >
              <p>
                No contents yet
              </p>
            </div>
            <div
              :class="{ 'mt-2': index > 0 }"
              v-for="(chat, index) in chatList"
              :key="index"
              ref="chatListElems"
            >
              <div
                class="d-flex"
                :class="{ 'justify-end': chat.role === 'user', 'mr-10': chat.role === 'assistant', 'ml-10': chat.role === 'user' }"
              >
                <p
                  class="pa-3 rounded markdown-body"
                  :class="chat.role"
                  v-html="chat.role === 'assistant' ? marked(chat.content) : chat.content"
                >
                </p>
              </div>
            </div>
            <div
              class="mt-2"
              v-if="sending"
            >
              <div
                class="d-flex"
              >
                <p
                  class="pa-3 rounded assistant"
                >
                  Waiting...
                  <v-progress-circular
                    :size="30"
                    color="white"
                    indeterminate
                  ></v-progress-circular>
                </p>
              </div>
            </div>
            <!--clear-->
            <div
              class="mt-2"
              v-if="chatList.filter((chat) => chat.role === 'error' && chat.content !== 'context_length_exceeded').length > 0"
            >
              <div
                class="d-flex justify-center"
              >
                <v-btn
                  color="error"
                  @click="chatList = []"
                >
                  Clear
                </v-btn>
              </div>
            </div>
          </v-list>
        </div>
      </v-card>
    </v-col>
  </v-row>

  <v-bottom-navigation>
    <v-card style="width: 100%; max-width: 900px;">
      <v-text-field
        v-model="content"
        label="content"
        single-line
        hide-details
        append-inner-icon="mdi-send"
        @click:append-inner="sendcontent"
        @keypress.enter="sendcontent"
      />
    </v-card>
  </v-bottom-navigation>
</template>

<style scoped>
.user {
  background-color: #5ee486;
}
.assistant {
  background-color: #383838;
  color: white;
}
.error {
  background-color: #ff0000;
  color: white;
}
</style>
