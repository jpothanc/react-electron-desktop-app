# Build Web Desktop Apps using Electron

Electron is a popular framework that allows developers to build cross-platform desktop applications using web technologies.

My goal here is to demonstrate how you can build your first Electron app using React and Vite on Windows with a simple working solution.

### Prerequisites

Node.js: Required for managing the development toolchain.

### Electron Stand-Alone App in 11 Steps

```c
///Install node.js and check thier versions
1. check node version using node -v

//Create react app called my-first-tauri-app using vite
2. npm create vite@latest react-electron-desktop-app -- --template react-ts
3. cd react-electron-desktop-app
4. npm install

//Electron: Framework for building cross-platform desktop apps
//with web technologies.
//Wait-on: CLI tool that waits for resources like files and ports
//to be available before proceeding.
//Concurrently: Allows running multiple commands simultaneously
//in the same terminal.
//Cross-env: Facilitates setting environment variables across
//different operating systems.
5. npm i electron wait-on concurrently cross-env

// Open the react app in VS Code
6. code .

7. create a electron folder and create preload.js (keep this empty
  and electron.js (see electron.js contents section)  files

8. Edit package.json
  {
    "name": "react-electron-desktop-app",
    "private": true,
    "version": "0.0.0",
    "type": "module" //remove this
    "main": "electron/electron.js", // add this
    .....

//Edit vite.config.ts to override the port to 3000
9. update scripts section as below
   "scripts": {
    "dev": "vite",
    "build": "tsc --noEmit && vite build",
    "serve": "vite preview",
    "electron": "wait-on tcp:3001 && cross-env IS_DEV=true NODE_ENV=development electron .",
    "electron:pack": "electron-builder --dir",
    "electron:dev": "concurrently -k \"cross-env BROWSER=none npm run dev\" \"npm run electron\"",
    "electron:builder": "electron-builder",
    "build:for:electron": "tsc --noEmit && cross-env ELECTRON=true vite build",
    "app:build": "npm run build:for:electron && npm run electron:builder"
  },

//Edit vite.config.ts to override the port to 3001
10. By default react app runs on 5173, we need to override this to 3001
   To do this, open vite.config.ts and add the server and port options.
    export default defineConfig({
      plugins: [react()],
    //override port
      server:{
        port: 3001,
      }
    })

//bring up the standalone app
11. npm run electron:dev
```

electron.js contents

```javascript
const { app, BrowserWindow } = require("electron");
const path = require("path");

function createWindow() {
  const win = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      preload: path.join(__dirname, "preload.js"),
      nodeIntegration: true,
      contextIsolation: false,
    },
  });
  if (process.env.NODE_ENV == "development") {
    win.loadURL("http://localhost:3001");
    win.webContents.openDevTools();
  } else {
    console.log(
      `Development Mode: Loading from localhost = ${process.env.NODE_ENV}`
    );
    win.loadFile(path.join(__dirname, "../index.html"));
  }
}

app.whenReady().then(() => {
  createWindow();

  app.on("activate", () => {
    if (BrowserWindow.getAllWindows().length === 0) {
      createWindow();
    }
  });
});

app.on("window-all-closed", () => {
  if (process.platform !== "darwin") {
    app.quit();
  }
});
```
