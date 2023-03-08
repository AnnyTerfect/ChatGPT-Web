<script setup>
import { ref, onMounted, watch } from 'vue'
import { marked } from 'marked';

const content = ref('')
const chatList = ref([])
const chatListElems = ref(null)

function sendcontent(event) {
  // Shift + Enter
  if (event.shiftKey || content.value === '') return

  chatList.value.push({role: 'user', content: content.value })
  setTimeout(() => {
    content.value = ''
    chatListElems.value[chatListElems.value.length - 1].scrollIntoView({ behavior: 'smooth' })
  }, 0)

  const fetchStream = (reader) => {
    return reader.read().then((result) => {
      if (result.done) {
        return;
      }
      const values = new TextDecoder("utf-8").decode(result.value).trim();
      /*
      if (res.error) throw `${res.error.code}: ${res.error.message}`
      chatList.value.splice(responseIndex, 1, {role: 'assistant', content: res.choices[0].message.content })
      chatListElems.value[chatListElems.value.length - 1].scrollIntoView({ behavior: 'smooth' })
      setTimeout(() => {
        chatListElems.value[chatListElems.value.length - 1].scrollIntoView({ behavior: 'smooth' })
      }, 300)*/
      for (let value of values.split('\n')) {
        value = value.replace('data: ', '').trim()

        if (value && value !== '[DONE]' && value !== '{' && value !== '}') {
          let res = JSON.parse(value)
          if (res.error) throw `${res.error.code}: ${res.error.message}`

          let deltaContent = res.choices[0].delta.content
          if (deltaContent && deltaContent.trim()) {
            let oldContent = chatList.value[responseIndex].content
            chatList.value.splice(responseIndex, 1, {role: 'assistant', content: oldContent + deltaContent })
            chatListElems.value[chatListElems.value.length - 1].scrollIntoView({ behavior: 'smooth' })
            setTimeout(() => {
              chatListElems.value[chatListElems.value.length - 1].scrollIntoView({ behavior: 'smooth' })
            }, 300)
          }
        }
      }
      // console.log(JSON.parse(value.replace('data: ', '').trim()))
      return fetchStream(reader);
    });
  }

  let responseIndex = chatList.value.length
  chatList.value.push({role: 'sending', content: ''})
  const apiKey = localStorage.getItem('apiKey')
  fetch('https://api.openai.com/v1/chat/completions', {
    method: 'post',
    headers: {
      'Content-Type': 'application/json',
      Authorization: 'Bearer ' + apiKey,
    },
    body: JSON.stringify({
      model: 'gpt-3.5-turbo',
      messages: chatList.value.filter((chat) => chat.role == 'user' || chat.role == 'assistant'),
      stream: true,
    })
  })
  .then(async (res) => res.body.getReader())
  .then(fetchStream)
  .catch((err) => {
    console.log(err)
    chatList.value.splice(responseIndex, 1, {role: 'error', content: err })
  }) 
}

const inputTextField = ref(null)
onMounted(() => {
  inputTextField.value.focus()
})
</script>

<template>
  <v-row class="min-h-1/1" style="max-width: 900px; margin: auto">
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
                v-if="chat.role !== 'sending'"
              >
                <p
                  class="pa-3 rounded markdown-body overflow-scroll"
                  :class="chat.role"
                  v-html="chat.role === 'assistant' ? marked(chat.content) : chat.content"
                >
                </p>
              </div>
              
              <div
                class="d-flex"
                v-if="chat.role === 'sending'"
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
              v-if="chatList.filter((chat) => chat.role === 'error' && chat.content === 'context_length_exceeded').length > 0"
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
      <v-textarea
        ref="inputTextField"
        v-model="content"
        label="content"
        rows="1"
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
