# Electron Desktop App Migration Guide

## 🎯 Overview
Your Restaurant POS System has been successfully migrated to **Electron.js** - a cross-platform desktop application framework. This migration follows the **latest Electron.js best practices (2024-2026)** from the official documentation.

### Architecture
```
CLIENT'S COMPUTER (Restaurant)
┌─────────────────────────────────────┐
│  Habibi POS.exe (One App)           │
│  ├─ Electron Desktop App (Frontend) │
│  ├─ Express Backend (Auto-starts)   │
│  ├─ MongoDB Local Database          │
│  └─ Thermal Printer Access          │
└─────────────────────────────────────┘
```

## 🔒 Security Implementation

Your implementation follows **Electron's Security Checklist**:

### ✅ Implemented Security Features

1. **Context Isolation** (Default since Electron 12)
   - `contextIsolation: true` - Isolates preload scripts from renderer
   - Uses `contextBridge` API for secure communication
   - Prevents web content from accessing privileged APIs

2. **Process Sandboxing** (Default since Electron 20)
   - `sandbox: true` - Limits renderer process capabilities
   - Protects against malicious code execution
   - Uses OS-level security features

3. **Node.js Integration Disabled**
   - `nodeIntegration: false` - Prevents direct Node.js access from renderer
   - All Node.js functionality exposed via secure IPC channels

4. **Navigation Protection**
   - Blocks navigation to external URLs
   - Prevents opening new windows
   - Validates all web content creation

5. **Secure IPC Communication**
   - Uses `ipcMain.handle()` and `ipcRenderer.invoke()` pattern
   - All APIs exposed through `contextBridge.exposeInMainWorld()`
   - No direct exposure of Electron internals

## 📁 Project Structure

```
pos-frontend/
├── electron/
│   ├── main.js          # Main process (manages app lifecycle, spawns backend)
│   └── preload.js       # Preload script (secure IPC bridge)
├── src/                 # React application (renderer process)
├── dist/                # Production build output
├── public/
│   └── icon.png         # Application icon
└── package.json         # Electron configuration

pos-backend/             # Express server (spawned as child process)
```

## 🚀 How to Run

### Development Mode

**Option 1: Separate Terminals (Recommended)**
```bash
# Terminal 1: Start MongoDB
mongod

# Terminal 2: Start Backend
cd pos-backend
bun run dev

# Terminal 3: Start Electron App
cd pos-frontend
bun run electron:dev
```

**Option 2: All-in-One** (if you modify main.js to spawn backend in dev)
```bash
cd pos-frontend
bun run electron:dev
```

### Production Build

```bash
cd pos-frontend
bun run electron:build
```

**Output:**
- **Linux**: `release/Habibi POS-0.0.0.AppImage`
- **Windows**: `release/Habibi POS Setup 0.0.0.exe` (if built on Windows)

## 📦 What's Included in the Build

The `electron-builder` configuration packages:

1. **Frontend**: Built React app (`dist/` folder)
2. **Backend**: Entire `pos-backend` folder (excluding `node_modules`)
3. **Electron Runtime**: Chromium + Node.js
4. **Dependencies**: All production npm packages

### Backend Dependencies Handling

⚠️ **Important**: The backend's `node_modules` are **NOT** included by default. You need to:

**Option A**: Install backend dependencies during build
```bash
cd pos-backend && npm install --production
```
Then the build will include them.

**Option B**: Bundle backend with webpack/esbuild (advanced)

## 🔌 Using Electron APIs in React

Your React app can now access Electron features through the `window.electronAPI` object:

```javascript
// In any React component
const handlePrint = async () => {
  const result = await window.electronAPI.printReceipt({
    orderId: '123',
    items: [...],
    total: 1500
  });
  console.log('Print result:', result);
};

const checkBackend = async () => {
  const status = await window.electronAPI.getBackendStatus();
  console.log('Backend running:', status.running);
};

// Platform detection
const platform = window.electronAPI.platform; // 'win32', 'darwin', 'linux'
```

## 🖨️ Thermal Printer Integration (TODO)

The IPC handler is ready in `main.js`. To implement:

1. Install printer library: `npm install node-thermal-printer`
2. Update `main.js` IPC handler:
```javascript
const { ThermalPrinter } = require('node-thermal-printer');

ipcMain.handle('print-receipt', async (event, receiptData) => {
  const printer = new ThermalPrinter({
    type: 'epson',
    interface: 'tcp://192.168.1.100'
  });
  
  printer.println(receiptData.header);
  // ... format receipt
  await printer.execute();
  return { success: true };
});
```

## 🗄️ MongoDB Bundling (TODO)

Currently, the app expects MongoDB to be installed separately. For **zero-configuration**:

### Option 1: mongodb-memory-server (Recommended)
```bash
npm install mongodb-memory-server
```

In `main.js`:
```javascript
const { MongoMemoryServer } = require('mongodb-memory-server');

async function startDatabase() {
  const mongod = await MongoMemoryServer.create({
    instance: {
      dbPath: path.join(app.getPath('userData'), 'database'),
      storageEngine: 'wiredTiger'
    }
  });
  
  return mongod.getUri();
}
```

### Option 2: Bundle mongod binary
1. Download `mongod` for target platform
2. Add to `extraResources` in `package.json`
3. Spawn it before backend in `main.js`

## 🔧 Configuration

### Electron Builder (`package.json`)

```json
{
  "build": {
    "appId": "com.habibipos.app",
    "productName": "Habibi POS",
    "files": ["dist/**/*", "electron/**/*"],
    "extraResources": [
      {
        "from": "../pos-backend",
        "to": "backend",
        "filter": ["**/*", "!node_modules", "!package-lock.json"]
      }
    ],
    "win": {
      "target": "nsis",
      "icon": "public/icon.png"
    },
    "linux": {
      "target": "AppImage",
      "category": "Office"
    }
  }
}
```

### Vite Configuration

```javascript
export default defineConfig({
  plugins: [react()],
  base: './', // Critical for file:// protocol
});
```

## 🐛 Troubleshooting

### Issue: "Cannot find module 'electron'"
**Solution**: Run `bun install` in `pos-frontend`

### Issue: Backend not starting in production
**Solution**: 
1. Check backend dependencies are installed
2. Verify `extraResources` path in package.json
3. Check console logs in the packaged app

### Issue: Assets not loading
**Solution**: Ensure `base: './'` is set in `vite.config.js`

### Issue: CORS errors
**Solution**: Backend already configured to allow `file://` and `null` origins

## 📚 References

- [Electron Security](https://www.electronjs.org/docs/latest/tutorial/security)
- [Context Isolation](https://www.electronjs.org/docs/latest/tutorial/context-isolation)
- [Process Model](https://www.electronjs.org/docs/latest/tutorial/process-model)
- [IPC Communication](https://www.electronjs.org/docs/latest/tutorial/ipc)
- [electron-builder](https://www.electron.build/)

## 🎉 For Your Client

### Installation (One-time)
1. Download `Habibi-POS-Setup.exe` (or `.AppImage` for Linux)
2. Double-click to install
3. Desktop shortcut created automatically

### Daily Use
1. Double-click "Habibi POS" icon
2. App automatically starts:
   - MongoDB starts in background (once implemented)
   - Express server starts in background
   - React UI opens
3. Use the POS system normally
4. Print receipts (once printer is configured)
5. Close app when done (everything stops automatically)

**No internet needed! No technical setup!** ✨
