class AsyncEvent {
  constructor () {
    this.events = {}
    this.queue = {}
  }
  on (event, fn) {
    (this.events[event] || (this.events[event] = [])).push(fn)
  }
  off (event, fn) {
    let events = this.events
    if (events[event]) {
      let list = events[event]
      if (fn) {
        let pos = list.indexOf(fn)
        if (pos !== -1) {
          list.splice(pos, 1)
        }
      } else {
        delete events[event]
      }
    }
  }
  emit (event, ...args) {
    let events = this.events
    if (events[event]) {
      let list = events[event].slice()
      let fn
      while ((fn = list.shift())) {
        fn(...args)
      }
    }
  }
  emitAsync (event, ctx) {
    if (arguments.length > 2) {
      return Promise.reject(new Error('emitAsync only accepts 2 argument'))
    }
    let next = promiseNext()
    this.emit(event, ctx, next)
    return next()
  }
  once (event, fn) {
    let once = (...args) => {
      this.off(event, once)
      fn(...args)
    }
    this.on(event, once)
  }
  before (event, fn) {
    (this.events[event] || (this.events[event] = [])).unshift(fn)
  }
  subscribe (event, fn) {
    this.on(event, fn)
    return () => {
      this.off(event, fn)
    }
  }
  emitTickAwait (event, ...args) {
    let q = this.queue[event] || (this.queue[event] = {launch: false, list: []})
    let fn = ((event, ...args) => {
      return () => {
        this.emit(event, ...args)
      }
    })(event, ...args)
    if (q.launch) {
      this.emit(event, ...args)
    } else {
      q.list.push(fn)
    }
  }
  emitTickExec (event, status = true) {
    let q = this.queue[event]
    if (typeof q === 'object') {
      if (typeof q.launch === 'boolean') {
        q.launch = status
      }
      if (q.list instanceof Array && q.list.length > 0) {
        let fn
        while ((fn = q.list.shift())) {
          fn()
        }
      }
    }
  }
}

function promiseNext () {
  let promise = Promise.resolve()
  let called = false
  let next = (resolve, reject) => {
    if (resolve || reject) {
      return (promise = promise.then(resolve, reject))
    }
    if (called) {
      return Promise.reject(new Error('next can not be called twice'))
    }
    called = true
    return promise
  }
  return next
}

class draw extends AsyncEvent {
  constructor () {
    super()
    this.on('draw', (e) => {
      console.log('draw', e)
    })
  }
  emit1 () {
    this.emitTickAwait('draw', {dt: 'd'})
  }
  emit2 () {
    this.emitTickAwait('draw', {dt: 'chat'})
  }
  start () {
     this.emitTickExec('draw')
  }
  stop () {
    this.emitTickExec('draw', false)
  }
}
