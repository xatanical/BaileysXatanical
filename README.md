# ⚡️ Baileys-Erlangga - WhatsApp Bot Framework 2025 Edition

<p align="center">
  <img src="https://files.catbox.moe/fi0co3.jpeg" width="300" alt="Baileys Erlangga Logo" />
</p>

<p align="center">
  <a href="https://github.com/ErlanggaaXzzz/Baileys"><img src="https://img.shields.io/github/stars/ErlanggaaXzzz/Baileys?style=for-the-badge" alt="Stars"></a>
  <a href="https://www.npmjs.com/package/baileys-erlangga"><img src="https://img.shields.io/npm/v/baileys-erlangga?style=for-the-badge" alt="NPM"></a>
  <a href="https://github.com/ErlanggaaXzzz/Baileys/blob/main/LICENSE"><img src="https://img.shields.io/github/license/ErlanggaaXzzz/Baileys?style=for-the-badge" alt="License"></a>
</p>

---

**Baileys-Erlangga** adalah versi modifikasi dari [WhiskeySockets/Baileys](https://github.com/WhiskeySockets/Baileys), dirancang khusus untuk para developer bot WhatsApp di tahun 2025. Fokus utama versi ini adalah kestabilan pairing code, session auto-recovery, dan fitur tambahan eksklusif yang tidak tersedia di versi original.

---

## 🚀 Keunggulan Utama

- 🔒 **Pairing Kode Custom** — Pairing bot tanpa ribet dan full kendali
- 🔄 **Session Recovery Otomatis** — Tidak perlu login ulang setiap waktu
- 💡 **Support WhatsApp Business API** — Cocok untuk bot UMKM & bisnis besar
- ⚙️ **Modular & Siap Pakai** — Gampang diintegrasikan ke berbagai jenis bot
- 📱 **Multi-Device Compatible** — 100% jalan di WA MD versi terbaru
- 💬 **Dukungan Komunitas Developer** — Dari dev, untuk dev
- 💥 **Diuji Crash-Resistant** — Cocok untuk eksperimen bot tingkat lanjut

---

---
## Contoh Penggunaan
```bash
const {
  default: makeWASocket,
  useMultiFileAuthState,
  DisconnectReason,
  fetchLatestBaileysVersion
} = require('baileys-erlangga');

const question = (text) => {
    const rl = readline.createInterface({ 
        input: process.stdin, 
        output: process.stdout 
    });
    return new Promise((resolve) => { rl.question(text, resolve) });
}

// Fungsi Untuk Connet Ke Whatsapp

async function StartBot(usePairingCode = true) {    
	const {
		state,
		saveCreds
	} = await useMultiFileAuthState("session")
	const erlangga = makeWASocket({
		printQRInTerminal: !usePairingCode,
		syncFullHistory: true,
		markOnlineOnConnect: true,
		connectTimeoutMs: 60000,
		defaultQueryTimeoutMs: 0,
		keepAliveIntervalMs: 10000,
		generateHighQualityLinkPreview: true,
		patchMessageBeforeSending: (message) => {
			const requiresPatch = !!(
				message.buttonsMessage ||
				message.templateMessage ||
				message.listMessage
			);
			if (requiresPatch) {
				message = {
					viewOnceMessage: {
						message: {
							messageContextInfo: {
								deviceListMetadataVersion: 2,
								deviceListMetadata: {},
							},
							...message,
						},
					},
				};
			}

			return message;
		},
		version: (await (await fetch('https://raw.githubusercontent.com/ErlanggaaXzzz/Baileys/refs/heads/main/lib/Defaults/baileys-version.json')).json()).version,
		browser: ["Ubuntu", "Chrome", "20.0.04"],
		logger: pino({
			level: 'fatal'
		}),
		auth: {
			creds: state.creds,
			keys: makeCacheableSignalKeyStore(state.keys, pino().child({
				level: 'silent',
				stream: 'store'
			})),
		}
	});

```

## ⚖️ Lisensi

Lisensi MIT (Free to Use)
Silakan gunakan, modifikasi, dan distribusikan sesuka hati — dengan tetap mencantumkan atribusi kepada pembuat asli dan fork ini:

Copyright (c) 2025 Erlangga Official


## 📦 Instalasi

```bash
npm install baileys-erlangga@mod