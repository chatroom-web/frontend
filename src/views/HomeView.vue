<template>
  <div class="flex h-screen bg-gray-200 overflow-hidden">
    <ChatList :chats="chats" @selectChat="selectChat" :onlineUsers="onlineUsers" />
    <ChatWindow
      :selectedChat="selectedChat"
      :meState="meState"
      :selectnum="selectnum"
      :from="from"
      @sendMessage="sendMessage"
      @sendBingoInvitation="sendbingo"
      @acceptGame="acceptGame"
      @cancelGame="cancelGame"
      @change_who_game="change_who_game"
      @endgame="endgame"
    />
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import createWebSocket from '@/functions/websocket'
import { me } from '@/functions/auth'
import ChatList from '@/components/main/ChatList.vue'
import ChatWindow from '@/components/main/ChatWindow.vue'

const ws = createWebSocket(`wss://${import.meta.env.VITE_BACKEND}`)
let meState = ref(null)
let onlineUsers = ref(0)

onMounted(async () => {
  try {
    meState.value = await me()
  } catch (e) {
    window.location.href = '/'
  }
  online(meState.value)
})

const endgame = () => {
  const chat = chats.value.find((chat) => chat.id === selectedChatId.value)
  chat.game = false
  chat.megame = false
  selectnum.value = -1
}

const online = (user) => {
  let data = {
    type: 'online',
    ...user
  }
  ws.send(JSON.stringify(data))
}

const change_who_game = (data) => {
  selectedChat.value.megame = data
}

const sendbingo = () => {
  if (selectedChat.value.game) return
  ws.send(
    JSON.stringify({
      type: 'message_game',
      isAnnouncement: false,
      to: selectedChat.value.id,
      from: meState.value.id,
      timestamp: new Date().toISOString()
    })
  )
}

const sleep = (ms) => {
  return new Promise((resolve) => setTimeout(resolve, ms))
}

ws.onmessage = async(event) => {
  let data = JSON.parse(event.data)
  console.log(data)
  if (data.type === 'online') {
    ws.send(
      JSON.stringify({
        type: 'getonline'
      })
    )
  } else if (data.type === 'getonline') {
    chats.value = chats.value.filter((chat) => {
      if (chat.id === -1) {
        return true
      }
      data.usersMemberList.some((user) => user.id === chat.id)
    })
    data.usersMemberList.forEach((user) => {
      if (user.id === meState.value.id) return
      if (chats.value.find((chat) => chat.id === user.id)) return
      chats.value.push({
        id: user.id,
        game: false,
        megame: false,
        name: user.username,
        avatar: user.picture,
        lastMessage: '',
        messages: []
      })
    })
    onlineUsers.value = data.usersMemberList.length
  } else if (data.type === 'message') {
    // 應付多點登陸
    console.log(data)
    let chat = null
    if (data.from === meState.value.id) {
      chat = chats.value.find((chat) => chat.id === data.to)
      console.log(chats.value, chat)
      chat.messages.push({
        id: Date.now(),
        text: data.text,
        isAnnouncement: data.isAnnouncement,
        isGame: false,
        sender: 'me',
        image: data.image,
        timestamp: data.timestamp
      })
    } else {
      if (data.to === -1) {
        chat = chats.value.find((chat) => chat.id === -1)
      } else {
        chat = chats.value.find((chat) => chat.id === data.from)
      }
      console.log(data)
      console.log(chats.value)
      console.log(chats.value.find((chat) => chat.id === data.from))
      chat.messages.push({
        id: Date.now(),
        avatar: data.isAnnouncement == true ? -1 : chats.value.find((chat) => chat.id === data.from).avatar,
        text: data.text,
        isAnnouncement: data.isAnnouncement,
        isGame: false,
        sender: data.sender.username,
        image: data.image,
        timestamp: data.timestamp
      })
    }
    console.log(selectedChatId.value, chat.id)
    if (selectedChatId.value !== chat.id) chat.unread = true
    chat.lastMessage = data.text
  } else if (data.type == 'message_game') {
    console.log(data)
    console.log(chats.value)
    let chat = null
    const now = Number(data.time)
    if (data.from === meState.value.id) {
      chat = chats.value.find((chat) => chat.id === data.to)
      chat.messages.push({
        id: now,
        // avatar: data.to === -1 ? null : chats.value.find((chat) => chat.id === data.from).avatar,
        isAnnouncement: data.isAnnouncement,
        isGame: true,
        sender: 'me',
        image: data.image,
        timestamp: data.timestamp
      })
    } else {
      chat = chats.value.find((chat) => chat.id === data.from)
      chat.messages.push({
        id: now,
        avatar: data.to === -1 ? null : chats.value.find((chat) => chat.id === data.from).avatar,
        text: data.text,
        isAnnouncement: data.isAnnouncement,
        isGame: true,
        sender: data.sender.username,
        image: data.image,
        timestamp: data.timestamp
      })
    }
    if (selectedChatId.value !== chat.id) chat.unread = true
    chat.lastMessage = 'Bingo Game Invitation'
  } else if (data.type == 'delete_message_game') {
    let chat = null
    console.log(data.id)
    if (data.from === meState.value.id) {
      chat = chats.value.find((chat) => chat.id === data.to)
      chat.messages = chat.messages.filter((message) => message.id !== data.id)
    } else {
      chat = chats.value.find((chat) => chat.id === data.from)
      console.log(chat)
      chat.messages = chat.messages.filter((message) => message.id !== data.id)
    }
    if (chat.lastMessage === 'Bingo Game Invitation') chat.lastMessage === ''
  } else if (data.type == 'start_game') {
    alert('若獲得五條連線及贏得勝利，若一整條連線上皆為奇數或者偶數，該條連線翻倍計算')
    let chat = null
    if (data.from === meState.value.id) {
      chat = chats.value.find((chat) => chat.id === data.to)
      chat.messages = chat.messages.filter((message) => message.id !== data.id)
      chat.game = true
      chat.megame = false
      if (data.start == meState.value.id) {
        chat.megame = true
      }
    } else {
      chat = chats.value.find((chat) => chat.id === data.from)
      console.log(chat)
      chat.messages = chat.messages.filter((message) => message.id !== data.id)
      chat.game = true
      chat.megame = false
      if (data.start != chat.id) {
        chat.megame = true
      }
    }
  } else if (data.type === 'select_number') {
    selectnum.value = data.number
    from.value = data.from
  } else if (data.type === 'game_end') {
    if (data.win != null) {
      if (data.win == meState.value.id) {
        console.log('win')
        alert('恭喜贏得勝利')
      } else {
        console.log('lose')
        alert('很可惜輸了')
      }
    }
    if (data.win == null) {
      if (data.to != meState.value.id) {
        alert('您已離開遊戲')
      } else {
        alert('對方已離開遊戲')
      }
    }
    endgame()
  }
}

const selectnum = ref(-1)
const from = ref(-1)

const selectedChatId = ref(-1)
const chats = ref([
  {
    id: -1,
    name: 'Group',
    avatar: null,
    lastMessage: '',
    messages: []
  }
])
const selectedChat = computed(() => {
  return chats.value.find((chat) => chat.id === selectedChatId.value) || chats.value[0]
})

const selectChat = (id) => {
  selectedChatId.value = id
}

const sendMessage = (message) => {
  console.log(message)
  if (selectedChat.value) {
    ws.send(
      JSON.stringify({
        type: 'message',
        isAnnouncement: false,
        to: selectedChat.value.id,
        from: meState.value.id,
        image: message.image,
        text: message.text,
        timestamp: new Date().toISOString()
      })
    )
  }
}

const acceptGame = (data) => {
  ws.send(
    JSON.stringify({
      type: 'start_game',
      to: selectedChatId.value,
      from: meState.value.id,
      timestamp: new Date().toISOString()
    })
  )
}

const cancelGame = (data) => {
  ws.send(
    JSON.stringify({
      type: 'delete_message_game',
      id: data.id,
      to: selectedChatId.value,
      from: meState.value.id,
      timestamp: new Date().toISOString()
    })
  )
}
</script>

<style scoped>
body {
  font-family: 'Inter', sans-serif;
  background-color: #f8f9fa;
}
</style>
