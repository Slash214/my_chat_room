<script setup>
import { nextTick, onMounted, onUpdated, ref, watch } from 'vue'
import { io } from 'socket.io-client'
import BetterScroll from 'better-scroll'
import { imgList, nameList } from './contanst'

let list = ref([])
let len = ref(0)
let content = ref('')
let roomName = '5201314'
let wrapper = ref(null)
let scroll = ref(null)
let ance = ref(true)
let count = ref(0)

onMounted(() => {
  console.log(genRandAvatar())
  nextTick(() => {
    console.log('wrapper.value', wrapper.value)
    initScroll()
  })
})

const initScroll = () => {
  scroll.value = new BetterScroll(wrapper.value, {
    mouseWheel: true,
    probeType: 3,
  })

  scroll.value.on('scroll', position => {
    // console.log('正在滚动...')
  })

  scroll.value.on('mousewheelStart', () => {
    console.log('鼠标开始滚动')

    if (list.value.length > 6) {
      let item = JSON.parse(JSON.stringify(list.value))
      len.value = item?.length || 0
      console.log('我是获取的数组', len.value)
      ance.value = false
    }
  })
}

const genRandAvatar = str => {
  let idx = Math.floor(Math.random() * 20)
  console.error('idxidxidxidxidxidx', idx)
  let val = str === 'name' ? nameList[idx] : imgList[idx]
  return val
}

const socket = io('http://127.0.0.1:520', {
  transports: ['websocket', 'polling'],
})
socket.on('connect', () => {
  let data = { roomName }
  console.log('链接成功.....')
  socket.emit('joinRoom', data)
  console.log(socket.connected)
})

socket.on('clientMessage', data => {
  console.warn('服务器返回的', data)
})

socket.on('receive', data => {
  console.error('服务器收到信息了', data)
  list.value.push(data)
})

const sendMessage = () => {
  console.log(content.value)

  if (!content.value) return
  let data = {
    roomName,
    content: content.value,
    avatar: genRandAvatar('avatar'),
    nickname: genRandAvatar('name'),
  }
  list.value.push(data)
  socket.send(data)
  content.value = ''
  if (!ance.value) {
    content.value = ''
    count.value = 0
    ance.value = true
    let elm = document.getElementById(`child${list.value.length - 1}`)
    scroll.value.scrollToElement(elm, 200, false, true)
  } 

}

const enterFn = e => {
  console.log('enter', e)
  sendMessage()
}

const gotoBottom = () => {
  len.value = 0
  count.value = 0
  ance.value = true
  let elm = document.getElementById(`child${list.value.length - 1}`)
  scroll.value.scrollToElement(elm, 200, false, true)
}

watch(list.value, newVal => {
  console.error('newVal.length', newVal.length)

  if (newVal.length > 4 && ance.value) {
    // count.value = newVal.length > len.value ? newVal.length - len.value : 0
    // console.log('当前的新消息', count.value)

    nextTick(() => {
      let elm = document.getElementById(`child${newVal.length - 1}`)
      console.log('获取的DOM', elm)
      scroll.value.scrollToElement(elm, 200, false, true)
    })
  }

  if (!ance.value) {
    console.log('老的数组', len.value)
    console.log('新的数组', newVal.length)
    count.value = newVal.length - len.value
  }

  setTimeout(() => {
    scroll.value.refresh()
  }, 50)
})
</script>

<template>
  <div class="main">
    <div ref="wrapper" class="wrapper">
      <ul class="content">
        <li class="item" v-for="(item, key) in list" :id="'child' + key">
          <img class="avatar" :src="item.avatar" alt="" />
          <div class="info">
            <span class="nickname">{{ item.nickname || '默认昵称' }}</span>
            <p class="desc">{{ item.content || '暂无内容' }}</p>
            <div class="triangle"></div>
          </div>
        </li>
      </ul>
      <div class="down" v-if="!ance" @click="gotoBottom">
        <p>
          <span class="count">{{ count }}</span
          >条新消息
        </p>
        <img src="./assets/img/down.png" alt="" />
      </div>
      <div class="empty" v-if="!list.length">爱呵呵聊天室</div>
    </div>

    <div class="box">
      <el-input
        @keyup.enter.native="enterFn"
        placeholder="输入聊天内容"
        v-model="content"
      ></el-input>
      <el-button class="btn" @click="sendMessage" type="primary">发送</el-button>
    </div>
  </div>
</template>

<style scoped lang="scss">
.main {
  width: 700px;
  height: 700px;
  border-radius: 12px;
  padding: 20px;
  background-color: #fff;
  box-shadow: 0 0 6px 3px #999;
  .wrapper {
    height: 600px;
    overflow: hidden;
    position: relative;
    .content {
      .item {
        margin-bottom: 20px;
        text-align: justify;
        color: #ddd;
        display: flex;
        box-sizing: border-box;
        &:first-child {
          margin-top: 20px;
        }
        &:last-child {
          padding-bottom: 100px;
        }
        .avatar {
          display: block;
          width: 50px;
          height: 50px;
          border-radius: 50%;
          margin: 10px 20px 0 0;
          border: 1px solid #ddd;
        }
        .info {
          box-sizing: border-box;
          display: flex;
          flex-direction: column;
          font-size: 14px;
          position: relative;
          .nickname {
            font-size: 12px;
            color: #00b7a0;
          }
          .desc {
            margin: 0;
            padding: 12px;
            background-color: rgb(235, 231, 231);
            border-radius: 6px;
            color: #333;
            text-align: justify;
          }
          .triangle {
            position: absolute;
            left: -8px;
            top: 70%;
            transform: translate(0, -70%);
            width: 0;
            height: 0;
            border-right: 10px solid rgb(235, 231, 231);
            border-top: 10px solid transparent;
            border-bottom: 10px solid transparent;
          }
        }
      }
    }
    .down {
      position: absolute;
      bottom: 20px;
      right: 0;
      cursor: pointer;
      display: flex;
      align-items: center;
      img {
        width: 30px;
        height: 30px;
        object-fit: cover;
      }
      p {
        color: #888;
        font-size: 14px;
        .count {
          font-weight: 700;
          color: red;
        }
      }
    }
    .empty {
      position: absolute;
      left: 50%;
      top: 50%;
      transform: translate(-50%, -50%);
      color: #bbb;
      line-height: 1.5;
      letter-spacing: 10px;
      font-weight: 700;
      font-size: 36px;
    }
  }
  .box {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background-color: #fff;
    height: 60px;
    line-height: 60px;
    .btn {
      margin-left: 20px;
    }
  }
}
@media screen and (min-width: 300px) and (max-width: 700px) {
  .main {
    width: 300px;
  }
}
</style>
