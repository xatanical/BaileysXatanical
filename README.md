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

async function connectBot() {
  const { state, saveCreds } = await useMultiFileAuthState('./session');
  const { version } = await fetchLatestBaileysVersion();

  const sock = makeWASocket({
    version,
    auth: state,
    printQRInTerminal: true
  });

  sock.ev.on('creds.update', saveCreds);
  sock.ev.on('connection.update', (update) => {
    const { connection, lastDisconnect } = update;
    if (
      connection === 'close' &&
      lastDisconnect?.error?.output?.statusCode !== DisconnectReason.loggedOut
    ) {
      connectBot(); // Reconnect otomatis
    } else if (connection === 'open') {
      console.log('✅ Bot sudah tersambung ke WhatsApp!');
    }
  });

  sock.ev.on('messages.upsert', async (msg) => {
    const m = msg.messages[0];
    if (!m.message || m.key.fromMe) return;

    const text =
      m.message.conversation ||
      m.message.extendedTextMessage?.text ||
      '';

    if (text.toLowerCase() === 'ping') {
      await sock.sendMessage(m.key.remoteJid, { text: 'Pong!' });
    }
  });
}

connectBot();
```

## ⚖️ Lisensi

Lisensi MIT (Free to Use)
Silakan gunakan, modifikasi, dan distribusikan sesuka hati — dengan tetap mencantumkan atribusi kepada pembuat asli dan fork ini:

Copyright (c) 2025 Erlangga Official


## 📦 Instalasi

```bash
npm install baileys-erlangga@mod