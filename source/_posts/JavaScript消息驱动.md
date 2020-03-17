---
title: JavaScript消息驱动
date: 2020-03-17 17:55:55
header_img : https://ss0.bdstatic.com/70cFvHSh_Q1YnxGkpoWK1HF6hhy/it/u=263690368,305441804&fm=26&gp=0.jpg
categories : JavaScript基础
discription : 'Javascript实现消息驱动'
---
# JavaScript消息驱动源码
```javascript
export default class EventWait {
  constructor () {
    // 消息驱动触发的事件对象
    this.eventHandler = {}
    // 等待的消息队列
    this.waitingList = []
    this._this = this
  }
  waitFor (waitList, handler, thisObj, ...parmas) {
    if (typeof resources === 'string') waitList = [waitList]
    if (waitList.length === 0) {
      this.runHandler(handler, parmas)
      return
    }
    if (thisObj) this._this = thisObj
    this.eventHandler = {
      'waiting': waitList.slice(0),
      'handler': handler,
      'parmas': parmas
    }
    this.waitingList = waitList
  }
  // 消息驱动完成后的事件调用
  runHandler (handler, parmas) {
    if (typeof handler === 'function') {
      // 将事件放入下一个事件队列中执行，否则会影响当前的事件执行顺序
      setTimeout(() => {
        handler.apply(this._this, parmas)
      }, 0)
    }
  }
  // 外部调用的触发消息驱动方法（通知）
  trigger (waitItem) {
    if (typeof waitItem === 'string') waitItem = [waitItem]
    for (let i = 0; i < waitItem.length; i++) {
      let wait = waitItem[i]
      if (!this.waitingList.includes(wait)) return
      this.release(wait)
    }
  }
  // 释放消息驱动队列
  release (waitItem) {
    if (typeof waitItem === 'string') waitItem = [waitItem]
    for (let i = 0; i < waitItem.length; i++) {
      if (this.eventHandler.waiting.includes(waitItem[i])) {
        let pos = this.eventHandler.waiting.indexOf(waitItem[i])
        this.eventHandler.waiting.splice(pos, 1)
        if (this.eventHandler.waiting.length === 0) {
          this.runHandler(this.eventHandler.handler, this.eventHandler.parmas)
        }
      }
    }
  }
}


```