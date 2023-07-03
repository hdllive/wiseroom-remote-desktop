<template>
  <div>
    <img ref="imageRef" class="remote" src="" alt="">
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import ReconnectingWebSocket from 'reconnecting-websocket'

onMounted(() => {
  watchControl()
  // createdWebsocket()
})

let socket = null
const scoketURL = 'ws://192.168.1.112:10086/Echo'
const imageRef = ref()
let timer = null
const clientWidth = document.documentElement.clientWidth
const clientHeight = document.documentElement.clientHeight
const scrollWidth = document.documentElement.scrollWidth
const scrollHeight = document.documentElement.scrollHeight
let imgWidth = null // 图片宽度
let imgHeight = null // 图片高度

const createdWebsocket = () => {
  socket = new ReconnectingWebSocket(scoketURL)
  socket.onopen = function () {
    console.log('连接已建立')
    resetHeart()
  }
  socket.onmessage = function (event) {
    if (event.data instanceof Blob) { // 处理桌面流
      const data = event.data
      const blob = new Blob([data], { type: "image/jpg" })
      imageRef.value.src = window.URL.createObjectURL(blob)
    } else {
      const objectDta = JSON.parse(event.data)
      console.log(objectDta)
      if (objectDta.type === 100) {
        resetHeart()
      }
    }
  }
  socket.onclose = function () {
    console.log('断开连接')
  }
}

const watchControl = () => { // 监听事件
  console.dir(imageRef.value)
  imgWidth = imageRef.value.clientWidth // 图片宽度
  imgHeight = imageRef.value.clientHeight // 图片高度
  console.log(imgWidth)
  console.log(imgHeight)

  window.ondragstart = function (e) { // 移除拖动事件
    e.preventDefault()
  }
  window.ondragend = function (e) { // 移除拖动事件
    e.preventDefault()
  }
  window.onkeydown = function (e) { // 键盘按下
    console.log('键盘按下', e)
    socket.send(JSON.stringify({ type: 0, key: e.keyCode }))
  }
  window.onkeyup = function (e) { // 键盘抬起
    console.log('键盘抬起', e)
    socket.send(JSON.stringify({ type: 1, key: e.keyCode }))
  }
  window.onmousedown = function (e) { // 鼠标单击按下
    console.log('单击按下', e)
    socket.send(JSON.stringify({ type: 5, x: e.pageX, y: e.pageY }))
  }
  window.onmouseup = function (e) { // 鼠标单击抬起
    console.log('单击抬起', e)
    socket.send(JSON.stringify({ type: 6, x: e.pageX, y: e.pageY }))
  }
  window.oncontextmenu = function (e) { // 鼠标右击
    console.log('右击', e)
    e.preventDefault()
    socket.send(JSON.stringify({ type: 4, x: e.pageX, y: e.pageY }))
  }
  window.ondblclick = function (e) { // 鼠标双击
    console.log('双击', e)
  }
  window.onmousewheel = function (e) { // 鼠标滚动
    console.log('滚动', e)
    const moving = e.deltaY / e.wheelDeltaY
    socket.send(JSON.stringify({ type: 7, x: e.x, y: e.y, deltaY: e.deltaY, deltaFactor: moving }))
  }
  window.onmousemove = function (e) { // 鼠标移动
    if (timer) {
      return
    } else {
      timer = setTimeout(function () {
        console.log(scrollWidth)
        console.log(scrollHeight)
        const percentageX = ((e.pageX / clientWidth) * 100).toFixed(2)
        const percentageY = ((e.pageY / clientHeight) * 100).toFixed(2)
        const newPageX = ''
        const newPageY = ''
        // console.log("鼠标移动:X轴位置" + e.pageX + ";Y轴位置:" + e.pageY)
        // console.log("百分比:X轴位置" + percentageX + "%" + ";Y轴位置:" + percentageY + "%")
        socket.send(JSON.stringify({ type: 2, x: e.pageX, y: e.pageY }))
        timer = null
      }, 60)
    }
  }
}

let heartTime = null // 心跳定时器实例
let socketHeart = 0  // 心跳次数
let HeartTimeOut = 20000 // 心跳超时时间
let socketError = 0 // 错误次数
// socket 重置心跳
const resetHeart = () => {
  socketHeart = 0
  socketError = 0
  clearInterval(heartTime)
  sendSocketHeart()
}
const sendSocketHeart = () => {
  heartTime = setInterval(() => {
    if (socketHeart <= 3) {
      console.log('心跳发送：', socketHeart)
      socket.send(
        JSON.stringify({
          type: 100,
          key: 'ping'
        })
      )
      socketHeart = socketHeart + 1
    } else {
      reconnect()
    }
  }, HeartTimeOut)
}
// socket重连
const reconnect = () => {
  socket.close()
  if (socketError <= 3) {
    clearInterval(heartTime)
    // createdWebsocket()
    socketError = socketError + 1
    console.log('socket重连', socketError)
  } else {
    console.log('重试次数已用完的逻辑', socketError)
    clearInterval(heartTime)
  }
}
</script>

<style scoped>
.remote {
  /* width: 100%; */
  /* height: v-bind(clientHeight); */
}
</style>
