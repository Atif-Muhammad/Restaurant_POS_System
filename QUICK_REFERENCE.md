# 🚀 Quick Reference Card

## Development Commands

```bash
# Start MongoDB
mongod

# Start Backend (Terminal 1)
cd pos-backend && bun run dev

# Start Electron App (Terminal 2)
cd pos-frontend && bun run electron:dev

# Build for Production
cd pos-frontend && bun run electron:build
```

## Electron API Usage

```javascript
// In any React component

// 1. Check if running in Electron
const isElectron = window.electronAPI !== undefined;

// 2. Print Receipt
await window.electronAPI.printReceipt({
  orderId: 'ORD-123',
  items: [{ name: 'Item', quantity: 1, price: 100 }],
  total: 100
});

// 3. Check Backend Status
const status = await window.electronAPI.getBackendStatus();
// Returns: { running: true, port: 3000 }

// 4. Platform Detection
const platform = window.electronAPI.platform;
// Returns: 'win32' | 'darwin' | 'linux'
```

## File Structure

```
pos-frontend/
├── electron/
│   ├── main.js          ← Main process
│   └── preload.js       ← IPC bridge
├── src/
│   ├── components/
│   │   └── ElectronExample.jsx  ← Usage example
│   └── types/
│       └── electron.d.ts        ← TypeScript types
└── package.json         ← Electron config

pos-backend/             ← Express server
```

## Security Features (Enabled)

- ✅ Context Isolation
- ✅ Process Sandboxing  
- ✅ Node.js Integration Disabled
- ✅ Navigation Protection
- ✅ Secure IPC via contextBridge

## Build Output

```
release/
└── Habibi POS-0.0.0.AppImage    (Linux)
└── Habibi POS Setup 0.0.0.exe   (Windows)
```

## Next Steps

1. ⚠️ Install backend dependencies: `cd pos-backend && npm install --production`
2. ⚠️ Implement MongoDB bundling (see ELECTRON_SETUP.md)
3. 📝 Add thermal printer integration
4. 🎨 Add application icon (512x512 PNG)

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "Cannot find module 'electron'" | Run `bun install` in pos-frontend |
| Backend not starting | Install backend dependencies |
| Assets not loading | Check `base: './'` in vite.config.js |
| CORS errors | Already fixed in backend |

## Documentation

- 📖 `ELECTRON_SETUP.md` - Complete guide
- 📝 `MIGRATION_SUMMARY.md` - What's been done
- 💡 `src/components/ElectronExample.jsx` - Code examples

---
**Status**: ✅ Ready for development
**Version**: Electron 40.1.0
