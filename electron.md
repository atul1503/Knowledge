# Electron - Desktop Apps with Web Technologies

Notes:

1. **Electron** lets you build desktop apps using HTML, CSS, and JavaScript. Think of it as running a web browser that only shows your app. Popular apps like VS Code, Discord, Slack, and WhatsApp Desktop are built with Electron.

2. Every Electron app has two main parts:
   - **Main process**: Controls the app lifecycle, creates windows, handles system events
   - **Renderer process**: Displays the UI (HTML/CSS/JS), one for each window
   ```javascript
   // main.js - This runs in the main process
   const { app, BrowserWindow } = require('electron')
   
   function createWindow() {
       const win = new BrowserWindow({
           width: 800,    // window width in pixels
           height: 600,   // window height in pixels
           webPreferences: {
               nodeIntegration: true  // allows Node.js in renderer
           }
       })
       win.loadFile('index.html')  // loads your HTML file
   }
   
   app.whenReady().then(createWindow)  // create window when app is ready
   ```

3. **Basic project structure** for an Electron app:
   ```
   my-electron-app/
   ├── package.json          // app metadata and dependencies
   ├── main.js              // main process entry point
   ├── index.html           // your app's HTML
   ├── style.css            // your app's CSS
   └── renderer.js          // renderer process JavaScript
   ```

4. **package.json** must specify Electron as the main entry point:
   ```json
   {
     "name": "my-app",
     "version": "1.0.0",
     "main": "main.js",        // tells Electron which file to run first
     "scripts": {
       "start": "electron .",   // npm start will launch your app
       "build": "electron-builder"
     },
     "devDependencies": {
       "electron": "^27.0.0"   // Electron framework
     }
   }
   ```

## How to create your first Electron app

1. **Set up the project**:
   ```bash
   mkdir my-electron-app
   cd my-electron-app
   npm init -y
   npm install electron --save-dev
   ```

2. **Create main.js** (main process):
   ```javascript
   const { app, BrowserWindow } = require('electron')
   const path = require('path')
   
   function createWindow() {
     const mainWindow = new BrowserWindow({
       width: 1200,
       height: 800,
       webPreferences: {
         preload: path.join(__dirname, 'preload.js'),  // preload script runs before renderer
         contextIsolation: true,    // security feature, isolates main and renderer
         nodeIntegration: false     // security best practice
       }
     })
   
     mainWindow.loadFile('index.html')
     
     // Open DevTools in development
     // mainWindow.webContents.openDevTools()
   }
   
   // App event listeners
   app.whenReady().then(createWindow)    // create window when app starts
   
   app.on('window-all-closed', () => {   // quit when all windows closed
     if (process.platform !== 'darwin') app.quit()  // except on macOS
   })
   
   app.on('activate', () => {            // create window when dock icon clicked (macOS)
     if (BrowserWindow.getAllWindows().length === 0) createWindow()
   })
   ```

3. **Create index.html** (your app's UI):
   ```html
   <!DOCTYPE html>
   <html>
   <head>
     <meta charset="UTF-8">
     <title>My Electron App</title>
     <link rel="stylesheet" href="style.css">
   </head>
   <body>
     <h1>Hello from Electron!</h1>
     <button id="myButton">Click me</button>
     <p id="output"></p>
     
     <script src="renderer.js"></script>
   </body>
   </html>
   ```

4. **Create renderer.js** (runs in the renderer process):
   ```javascript
   // This code runs in the browser-like renderer process
   document.getElementById('myButton').addEventListener('click', () => {
     document.getElementById('output').textContent = 'Button clicked!'
   })
   
   // You can use modern web APIs here
   console.log('Renderer process started')
   ```

5. **Run your app**:
   ```bash
   npm start
   ```

## Communication between main and renderer processes

Since main and renderer processes are separate, they need to communicate through **IPC (Inter-Process Communication)**.

### Sending messages from renderer to main

1. **Create preload.js** (bridge between main and renderer):
   ```javascript
   // preload.js - runs in renderer but has access to Node.js APIs
   const { contextBridge, ipcRenderer } = require('electron')
   
   // Expose protected methods that allow the renderer process to use
   // the ipcRenderer without exposing the entire object
   contextBridge.exposeInMainWorld('electronAPI', {
     openFile: () => ipcRenderer.invoke('dialog:openFile'),        // call main process
     sendMessage: (message) => ipcRenderer.invoke('send-message', message),
     onUpdateCounter: (callback) => ipcRenderer.on('update-counter', callback)
   })
   ```

2. **Update main.js** to handle IPC messages:
   ```javascript
   const { app, BrowserWindow, ipcMain, dialog } = require('electron')
   
   // Handle IPC messages from renderer
   ipcMain.handle('dialog:openFile', async () => {
     const { canceled, filePaths } = await dialog.showOpenDialog()  // opens file dialog
     if (canceled) {
       return
     } else {
       return filePaths[0]  // return selected file path
     }
   })
   
   ipcMain.handle('send-message', async (event, message) => {
     console.log('Received from renderer:', message)
     return `Echo: ${message}`  // return response to renderer
   })
   ```

3. **Use IPC in renderer.js**:
   ```javascript
   // renderer.js - now can communicate with main process
   document.getElementById('openFile').addEventListener('click', async () => {
     const filePath = await window.electronAPI.openFile()  // calls main process
     if (filePath) {
       document.getElementById('filePath').textContent = filePath
     }
   })
   
   document.getElementById('sendMessage').addEventListener('click', async () => {
     const response = await window.electronAPI.sendMessage('Hello from renderer!')
     console.log('Main process replied:', response)
   })
   ```

### Sending messages from main to renderer

```javascript
// In main.js - send data to renderer
function createWindow() {
  const mainWindow = new BrowserWindow({ /* ... */ })
  
  // Send message to renderer after 3 seconds
  setTimeout(() => {
    mainWindow.webContents.send('update-counter', { count: 42 })
  }, 3000)
}

// In renderer.js - listen for messages from main
window.electronAPI.onUpdateCounter((event, data) => {
  document.getElementById('counter').textContent = data.count
})
```

## Working with the file system

Electron apps can access the file system through Node.js APIs in the main process.

1. **Reading files** (in main process):
   ```javascript
   const fs = require('fs').promises
   const path = require('path')
   
   ipcMain.handle('read-file', async (event, filePath) => {
     try {
       const data = await fs.readFile(filePath, 'utf8')  // read file as text
       return { success: true, data }
     } catch (error) {
       return { success: false, error: error.message }
     }
   })
   
   ipcMain.handle('write-file', async (event, filePath, content) => {
     try {
       await fs.writeFile(filePath, content, 'utf8')   // write text to file
       return { success: true }
     } catch (error) {
       return { success: false, error: error.message }
     }
   })
   ```

2. **Using file operations in renderer**:
   ```javascript
   // In preload.js
   contextBridge.exposeInMainWorld('electronAPI', {
     readFile: (filePath) => ipcRenderer.invoke('read-file', filePath),
     writeFile: (filePath, content) => ipcRenderer.invoke('write-file', filePath, content)
   })
   
   // In renderer.js
   async function loadFile() {
     const result = await window.electronAPI.readFile('/path/to/file.txt')
     if (result.success) {
       document.getElementById('fileContent').textContent = result.data
     } else {
       console.error('File read error:', result.error)
     }
   }
   ```

## Building and packaging your app

1. **Install electron-builder** for packaging:
   ```bash
   npm install electron-builder --save-dev
   ```

2. **Add build scripts to package.json**:
   ```json
   {
     "scripts": {
       "start": "electron .",
       "build": "electron-builder",
       "build:win": "electron-builder --win",
       "build:mac": "electron-builder --mac", 
       "build:linux": "electron-builder --linux"
     },
     "build": {
       "appId": "com.mycompany.myapp",
       "productName": "My Electron App",
       "directories": {
         "output": "dist"          // where built apps go
       },
       "files": [
         "main.js",
         "preload.js", 
         "renderer.js",
         "index.html",
         "style.css",
         "package.json"
       ],
       "mac": {
         "category": "public.app-category.productivity"
       },
       "win": {
         "target": "nsis"          // Windows installer type
       }
     }
   }
   ```

3. **Build your app**:
   ```bash
   npm run build        # builds for current platform
   npm run build:win    # builds Windows installer
   npm run build:mac    # builds macOS app
   ```

## Common Electron APIs and patterns

### App menu and context menus

```javascript
// In main.js
const { Menu } = require('electron')

const template = [
  {
    label: 'File',
    submenu: [
      {
        label: 'New',
        accelerator: 'CmdOrCtrl+N',    // keyboard shortcut
        click: () => {
          // Handle new file
          console.log('New file clicked')
        }
      },
      { type: 'separator' },           // menu divider
      {
        label: 'Exit',
        accelerator: process.platform === 'darwin' ? 'Cmd+Q' : 'Ctrl+Q',
        click: () => {
          app.quit()
        }
      }
    ]
  }
]

const menu = Menu.buildFromTemplate(template)
Menu.setApplicationMenu(menu)  // sets the app menu
```

### System notifications

```javascript
// In main.js
const { Notification } = require('electron')

function showNotification(title, body) {
  new Notification({ 
    title: title, 
    body: body,
    icon: 'path/to/icon.png'  // optional icon
  }).show()
}

// Usage
ipcMain.handle('show-notification', (event, title, body) => {
  showNotification(title, body)
})
```

### System tray

```javascript
// In main.js
const { Tray } = require('electron')

let tray = null

function createTray() {
  tray = new Tray('path/to/tray-icon.png')  // small icon for system tray
  
  const contextMenu = Menu.buildFromTemplate([
    { label: 'Show App', click: () => mainWindow.show() },
    { label: 'Quit', click: () => app.quit() }
  ])
  
  tray.setToolTip('My Electron App')      // hover text
  tray.setContextMenu(contextMenu)        // right-click menu
}
```

### Auto-updater

```javascript
// Install first: npm install electron-updater
const { autoUpdater } = require('electron-updater')

app.whenReady().then(() => {
  autoUpdater.checkForUpdatesAndNotify()  // checks for updates automatically
})

autoUpdater.on('update-available', () => {
  // Show notification that update is available
})

autoUpdater.on('update-downloaded', () => {
  // Show notification that update is ready to install
  autoUpdater.quitAndInstall()  // restart app with new version
})
```

## Security best practices

**Electron apps run web code with system access, so security is critical.**

1. **Always use contextIsolation and disable nodeIntegration**:
   ```javascript
   // In main.js
   new BrowserWindow({
     webPreferences: {
       contextIsolation: true,     // isolates main and renderer contexts
       nodeIntegration: false,     // prevents renderer from accessing Node.js directly
       preload: path.join(__dirname, 'preload.js')  // safe bridge between processes
     }
   })
   ```

2. **Validate all IPC messages**:
   ```javascript
   // In main.js - always validate data from renderer
   ipcMain.handle('save-file', async (event, filePath, content) => {
     // Validate inputs
     if (typeof filePath !== 'string' || typeof content !== 'string') {
       throw new Error('Invalid parameters')
     }
     
     // Prevent path traversal attacks
     if (filePath.includes('..')) {
       throw new Error('Invalid file path')
     }
     
     // Now safe to use the data
     await fs.writeFile(filePath, content)
   })
   ```

3. **Don't load remote content without careful consideration**:
   ```javascript
   // Bad - can execute malicious code
   mainWindow.loadURL('https://untrusted-site.com')
   
   // Better - load local files
   mainWindow.loadFile('index.html')
   
   // If you must load remote content, disable Node integration
   const remoteWindow = new BrowserWindow({
     webPreferences: {
       nodeIntegration: false,
       contextIsolation: true,
       sandbox: true  // enables additional security restrictions
     }
   })
   ```

## Common gotchas and solutions

1. **App doesn't quit on Windows/Linux**: Use the window-all-closed event properly.
   ```javascript
   app.on('window-all-closed', () => {
     if (process.platform !== 'darwin') {  // don't quit on macOS
       app.quit()
     }
   })
   ```

2. **Renderer process can't access Node.js**: This is intentional for security. Use IPC with preload scripts.

3. **App icon not showing**: Set icon in multiple places.
   ```javascript
   // In main.js
   const win = new BrowserWindow({
     icon: path.join(__dirname, 'assets/icon.png')  // window icon
   })
   
   // In package.json build config
   "build": {
     "mac": {
       "icon": "assets/icon.icns"    // macOS icon
     },
     "win": {
       "icon": "assets/icon.ico"     // Windows icon
     }
   }
   ```

4. **DevTools showing in production**: Only open DevTools in development.
   ```javascript
   if (process.env.NODE_ENV === 'development') {
     mainWindow.webContents.openDevTools()
   }
   ```

5. **Large bundle size**: Electron apps are big because they include Chromium. Minimize by:
   - Only packaging necessary files
   - Using `electron-builder`'s compression
   - Avoiding unnecessary dependencies

## Development workflow tips

1. **Hot reload during development**:
   ```bash
   npm install electron-reload --save-dev
   ```
   ```javascript
   // Add to main.js for development
   if (process.env.NODE_ENV === 'development') {
     require('electron-reload')(__dirname)
   }
   ```

2. **Debug main process**: Use `--inspect` flag.
   ```json
   // In package.json
   "scripts": {
     "debug": "electron --inspect=5858 ."
   }
   ```

3. **Debug renderer process**: Use Chrome DevTools (Ctrl+Shift+I or Cmd+Option+I).

4. **Environment variables**:
   ```javascript
   // Check if in development
   const isDev = process.env.NODE_ENV === 'development'
   
   // Use different URLs for dev/prod
   if (isDev) {
     mainWindow.loadURL('http://localhost:3000')  // dev server
   } else {
     mainWindow.loadFile('build/index.html')      // built files
   }
   ```

**Key concepts to remember:**
- Main process = Node.js server, manages app lifecycle and creates windows
- Renderer process = Chromium browser, displays your web app UI  
- IPC = Inter-Process Communication, how main and renderer talk to each other
- Preload scripts = Safe bridge between main and renderer processes
- Security first = Always use contextIsolation and validate IPC messages


## Working with Native Features

Electron provides several built-in modules for accessing native system features. Here's how to work with common ones:

1. **Camera access** using `desktopCapturer`:
   ```javascript
   // In main.js, enable required permissions
   const mainWindow = new BrowserWindow({
     webPreferences: {
       nodeIntegration: true,
       contextIsolation: false // Note: In production, use contextIsolation with preload scripts
     }
   });

   // In renderer.js
   const { desktopCapturer } = require('electron')

   async function getVideoSources() {
     const sources = await desktopCapturer.getSources({ 
       types: ['window', 'screen', 'camera']
     });
     
     // Get camera source
     const videoSource = sources.find(source => source.name === 'Camera');
     
     // Use with WebRTC
     const stream = await navigator.mediaDevices.getUserMedia({
       video: {
         mandatory: {
           chromeMediaSource: 'desktop',
           chromeMediaSourceId: videoSource.id
         }
       }
     });
     
     // Display in video element
     const videoElement = document.querySelector('video');
     videoElement.srcObject = stream;
   }
   ```

2. **GPS/Location** using HTML5 Geolocation API:
   ```javascript
   // In renderer process
   if ('geolocation' in navigator) {
     navigator.geolocation.getCurrentPosition(
       (position) => {
         const latitude = position.coords.latitude;
         const longitude = position.coords.longitude;
         console.log(`Location: ${latitude}, ${longitude}`);
       },
       (error) => {
         console.error('Error getting location:', error);
       },
       {
         enableHighAccuracy: true, // Use GPS if available
         timeout: 5000,           // Time to wait for location
         maximumAge: 0            // Don't use cached position
       }
     );
   }
   ```

3. **File system access** using Node.js `fs` module:
   ```javascript
   // In preload.js - create safe bridge
   const { contextBridge, ipcRenderer } = require('electron');
   const fs = require('fs');

   contextBridge.exposeInMainWorld('fileSystem', {
     readFile: (path) => {
       return new Promise((resolve, reject) => {
         fs.readFile(path, 'utf8', (error, data) => {
           if (error) reject(error);
           else resolve(data);
         });
       });
     },
     writeFile: (path, data) => {
       return new Promise((resolve, reject) => {
         fs.writeFile(path, data, 'utf8', (error) => {
           if (error) reject(error);
           else resolve();
         });
       });
     }
   });

   // In renderer.js - use the safe bridge
   async function saveFile(path, content) {
     try {
       await window.fileSystem.writeFile(path, content);
       console.log('File saved successfully');
     } catch (error) {
       console.error('Error saving file:', error);
     }
   }
   ```

4. **System notifications** using Electron's `Notification` module:
   ```javascript
   // In main process
   const { Notification } = require('electron');

   function showNotification(title, body) {
     new Notification({
       title: title,
       body: body,
       icon: './path/to/icon.png' // Optional
     }).show();
   }

   // Usage
   showNotification('Download Complete', 'Your file has been saved');
   ```

Remember these security best practices:
- Always use `contextIsolation` and preload scripts in production
- Validate all input from renderer process
- Only expose necessary native APIs
- Use permissions carefully to prevent unauthorized access
- Consider using `sandbox: true` in BrowserWindow for additional security

