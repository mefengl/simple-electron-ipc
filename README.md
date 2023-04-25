# simple-electron-ipc

```js
// https://stackoverflow.com/a/71495848/14435416
// preload.js

// Import the necessary Electron components.
const contextBridge = require('electron').contextBridge;
const ipcRenderer = require('electron').ipcRenderer;

// White-listed channels.
const ipc = {
    'render': {
        // From render to main.
        'send': [
            'message:fromRender'
        ],
        // From main to render.
        'receive': [
            'message:toRender'
        ],
        // From render to main and back again.
        'sendReceive': []
    }
};

// Exposed protected methods in the render process.
contextBridge.exposeInMainWorld(
    // Allowed 'ipcRenderer' methods.
    'ipcRender', {
        // From render to main.
        send: (channel, args) => {
            let validChannels = ipc.render.send;
            if (validChannels.includes(channel)) {
                ipcRenderer.send(channel, args);
            }
        },
        // From main to render.
        receive: (channel, listener) => {
            let validChannels = ipc.render.receive;
            if (validChannels.includes(channel)) {
                // Deliberately strip event as it includes `sender`.
                ipcRenderer.on(channel, (event, ...args) => listener(...args));
            }
        },
        // From render to main and back again.
        invoke: (channel, args) => {
            let validChannels = ipc.render.sendReceive;
            if (validChannels.includes(channel)) {
                return ipcRenderer.invoke(channel, args);
            }
        }
    }
);
```

above code can't take advantage of auto-completion, so need to further improve it.

like below(not finished yet)

```js
const { contextBridge, ipcRenderer } = require('electron')

const validChannels = {
  // From render to main.
  'send': [],
  // From main to render.
  'receive': [],
  // From render to main and back again.
  'sendReceive': []
};

const easy_expose = (functionNames) => {
  const exposedFunctions = {};
  functionNames.forEach(functionName => {
    const trigger_id = functionName.replace(/[A-Z]/g, letter => `-${letter.toLowerCase()}`);
    exposedFunctions[functionName] = (...args) => ipcRenderer.send(trigger_id, ...args);
  });

  contextBridge.exposeInMainWorld('electronAPI', exposedFunctions);
};

easy_expose(validChannels.send);
```
