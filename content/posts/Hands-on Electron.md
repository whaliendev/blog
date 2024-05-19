---
author: "Hwa"
title: "Hands on Electron"
date: "2022-11-05"
tags:
    - Electron
    - Node.js
    - JavaScript
summary: "Electron is a framework for creating native applications with web technologies like JavaScript, HTML, and CSS. It takes care of the hard parts so you can focus on the core of your application. This article will show you the main components and how they interact with each other."
---

## Hands on Electron

### Arch

Main(Node) + Renderer(Chrome)

+ `main.js`: the entry point of electron 

+ `index.html`: the main page of app, it will be loaded into renderer process

+ `renderer.js`: This file is loaded via the <script> tag in the index.html file and will be executed in the renderer process for that window. No Node.js APIs are available in this process because `nodeIntegration` is turned off and `contextIsolation` is turned on. Use the contextBridge API in `preload.js` to expose Node.js functionality from the main process.

+ `preload.js`: it's a script loaded before `index.html` by renderer process, like Chrome extension content script.  

preload.js is default sandboxed

<table><thead><tr><th>可用的 API</th><th>详细信息</th></tr></thead><tbody><tr><td>Electron 模块</td><td>渲染进程模块</td></tr><tr><td>Node.js 模块</td><td><a href="https://nodejs.org/api/events.html" target="_blank" rel="noopener noreferrer"><code>events</code></a>、<a href="https://nodejs.org/api/timers.html" target="_blank" rel="noopener noreferrer"><code>timers</code></a>、<a href="https://nodejs.org/api/url.html" target="_blank" rel="noopener noreferrer"><code>url</code></a></td></tr><tr><td>Polyfilled 的全局模块</td><td><a href="https://nodejs.org/api/buffer.html" target="_blank" rel="noopener noreferrer"><code>Buffer</code></a>、<a href="/zh/docs/latest/api/process"><code>process</code></a>、<a href="https://nodejs.org/api/timers.html#timers_clearimmediate_immediate" target="_blank" rel="noopener noreferrer"><code>clearImmediate</code></a>、<a href="https://nodejs.org/api/timers.html#timers_setimmediate_callback_args" target="_blank" rel="noopener noreferrer"><code>setImmediate</code></a></td></tr></tbody></table>



### Process Communication

+ `preload.js` ==> `renderer.js`

use contextBridge API to define global vars 

```js
const { contextBridge } = require('electron')

contextBridge.exposeInMainWorld('versions', {
  node: () => process.versions.node,
  chrome: () => process.versions.chrome,
  electron: () => process.versions.electron,
  // 能暴露的不仅仅是函数，我们还可以暴露变量
})
```

+ IPC

communicate between main process and renderer process, use IPC module(`ipcMain` + `ipcRenderer`)

+ `renderer.js` ==> `main.js`

first, expose `invoke` call to renderer.js in preload.js:

```js
const { contextBridge, ipcRenderer } = require('electron')

contextBridge.exposeInMainWorld('versions', {
  node: () => process.versions.node,
  chrome: () => process.versions.chrome,
  electron: () => process.versions.electron,
  ping: () => ipcRenderer.invoke('ping'),
  // 能暴露的不仅仅是函数，我们还可以暴露变量
})
```

second, register event handler in `main.js`:

```js
const { app, BrowserWindow, ipcMain } = require('electron')
const path = require('path')

const createWindow = () => {
  const win = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      preload: path.join(__dirname, 'preload.js'),
    },
  })
  ipcMain.handle('ping', () => 'pong')
  win.loadFile('index.html')
}
app.whenReady().then(createWindow)
```

finally, use `invoke` call in your renderer.js:

```js
const func = async () => {
  const response = await window.versions.ping()
  console.log(response) // 打印 'pong'
}

func()
```

