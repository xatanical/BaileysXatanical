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


## ⚖️ Lisensi

Lisensi MIT (Free to Use)
Silakan gunakan, modifikasi, dan distribusikan sesuka hati — dengan tetap mencantumkan atribusi kepada pembuat asli dan fork ini:

Copyright (c) 2025 Erlangga Official



## 📦 Instalasi

```bash
npm install baileys-erlangga@mod

```




---
## Contoh Penggunaan

```ts
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

### Emulating the Desktop app instead of the web

1. Baileys, by default, emulates a chrome web session
2. If you'd like to emulate a desktop connection (and receive more message history), add this to your Socket config:
    ``` ts
    const conn = makeWASocket({
        ...otherOpts,
        // can use Windows, Ubuntu here too
        browser: Browsers.macOS('Desktop'),
        syncFullHistory: true
    })
    ```

## Saving & Restoring Sessions

You obviously don't want to keep scanning the QR code every time you want to connect. 

So, you can load the credentials to log back in:
``` ts
import makeWASocket, { BufferJSON, useMultiFileAuthState } from 'baileys-erlangga'
import * as fs from 'fs'

// utility function to help save the auth state in a single folder
// this function serves as a good guide to help write auth & key states for SQL/no-SQL databases, which I would recommend in any production grade system
const { state, saveCreds } = await useMultiFileAuthState('auth_info_baileys')
// will use the given state to connect
// so if valid credentials are available -- it'll connect without QR
const conn = makeWASocket({ auth: state }) 
// this will be called as soon as the credentials are updated
conn.ev.on ('creds.update', saveCreds)
```

**Note:** When a message is received/sent, due to signal sessions needing updating, the auth keys (`authState.keys`) will update. Whenever that happens, you must save the updated keys (`authState.keys.set()` is called). Not doing so will prevent your messages from reaching the recipient & cause other unexpected consequences. The `useMultiFileAuthState` function automatically takes care of that, but for any other serious implementation -- you will need to be very careful with the key state management.

## Listening to Connection Updates

Baileys now fires the `connection.update` event to let you know something has updated in the connection. This data has the following structure:
``` ts
type ConnectionState = {
	/** connection is now open, connecting or closed */
	connection: WAConnectionState
	/** the error that caused the connection to close */
	lastDisconnect?: {
		error: Error
		date: Date
	}
	/** is this a new login */
	isNewLogin?: boolean
	/** the current QR code */
	qr?: string
	/** has the device received all pending notifications while it was offline */
	receivedPendingNotifications?: boolean 
}
```

**Note:** this also offers any updates to the QR


## Sending Messages

**Send all types of messages with a single function:**

### Non-Media Messages

``` ts
import { MessageType, MessageOptions, Mimetype } from 'baileys-erlangga'

const id = 'abcd@s.whatsapp.net' // the WhatsApp ID 
// send a simple text!
const sentMsg  = await sock.sendMessage(id, { text: 'oh hello there' })
// send a reply messagge
const sentMsg  = await sock.sendMessage(id, { text: 'oh hello there' }, { quoted: message })
// send a mentions message
const sentMsg  = await sock.sendMessage(id, { text: '@12345678901', mentions: ['12345678901@s.whatsapp.net'] })
// send a location!
const sentMsg  = await sock.sendMessage(
    id, 
    { location: { degreesLatitude: 24.121231, degreesLongitude: 55.1121221 } }
)
// send a contact!
const vcard = 'BEGIN:VCARD\n' // metadata of the contact card
            + 'VERSION:3.0\n' 
            + 'FN:Jeff Singh\n' // full name
            + 'ORG:Ashoka Uni;\n' // the organization of the contact
            + 'TEL;type=CELL;type=VOICE;waid=911234567890:+91 12345 67890\n' // WhatsApp ID + phone number
            + 'END:VCARD'
const sentMsg  = await sock.sendMessage(
    id,
    { 
        contacts: { 
            displayName: 'Jeff', 
            contacts: [{ vcard }] 
        }
    }
)

const reactionMessage = {
    react: {
        text: "💖", // use an empty string to remove the reaction
        key: message.key
    }
}

const sendMsg = await sock.sendMessage(id, reactionMessage)
```

### Sending messages with link previews

1. By default, WA MD does not have link generation when sent from the web
2. Baileys has a function to generate the content for these link previews
3. To enable this function's usage, add `link-preview-js` as a dependency to your project with `yarn add link-preview-js`
4. Send a link:
``` ts
// send a link
const sentMsg  = await sock.sendMessage(id, { text: 'Hi, this was sent using https://github.com/ErlanggaaXzz/Baileys' })
```

### Media Messages

Sending media (video, stickers, images) is easier & more efficient than ever. 
- You can specify a buffer, a local url or even a remote url.
- When specifying a media url, Baileys never loads the entire buffer into memory; it even encrypts the media as a readable stream.

``` ts
import { MessageType, MessageOptions, Mimetype } from 'baileys-erlangga'
// Sending gifs
await sock.sendMessage(
    id, 
    { 
        video: fs.readFileSync("Media/ma_gif.mp4"), 
        caption: "hello!",
        gifPlayback: true
    }
)

await sock.sendMessage(
    id, 
    { 
        video: "./Media/ma_gif.mp4", 
        caption: "hello!",
        gifPlayback: true
    }
)

// send an audio file
await sock.sendMessage(
    id, 
    { audio: { url: "./Media/audio.mp3" }, mimetype: 'audio/mp4' }
    { url: "Media/audio.mp3" }, // can send mp3, mp4, & ogg
)
```

### Notes

- `id` is the WhatsApp ID of the person or group you're sending the message to. 
    - It must be in the format ```[country code][phone number]@s.whatsapp.net```
	    - Example for people: ```+19999999999@s.whatsapp.net```. 
	    - For groups, it must be in the format ``` 123456789-123345@g.us ```. 
    - For broadcast lists, it's `[timestamp of creation]@broadcast`.
    - For stories, the ID is `status@broadcast`.
- For media messages, the thumbnail can be generated automatically for images & stickers provided you add `jimp` or `sharp` as a dependency in your project using `yarn add jimp` or `yarn add sharp`. Thumbnails for videos can also be generated automatically, though, you need to have `ffmpeg` installed on your system.
- **MiscGenerationOptions**: some extra info about the message. It can have the following __optional__ values:
    ``` ts
    const info: MessageOptions = {
        quoted: quotedMessage, // the message you want to quote
        contextInfo: { forwardingScore: 2, isForwarded: true }, // some random context info (can show a forwarded message with this too)
        timestamp: Date(), // optional, if you want to manually set the timestamp of the message
        caption: "hello there!", // (for media messages) the caption to send with the media (cannot be sent with stickers though)
        jpegThumbnail: "23GD#4/==", /*  (for location & media messages) has to be a base 64 encoded JPEG if you want to send a custom thumb, 
                                    or set to null if you don't want to send a thumbnail.
                                    Do not enter this field if you want to automatically generate a thumb
                                */
        mimetype: Mimetype.pdf, /* (for media messages) specify the type of media (optional for all media types except documents),
                                    import {Mimetype} from 'erlangga/baileys'
                                */
        fileName: 'somefile.pdf', // (for media messages) file name for the media
        /* will send audio messages as voice notes, if set to true */
        ptt: true,
        /** Should it send as a disappearing messages. 
         * By default 'chat' -- which follows the setting of the chat */
        ephemeralExpiration: WA_DEFAULT_EPHEMERAL
    }
    ```
## Forwarding Messages

``` ts
const msg = getMessageFromStore('455@s.whatsapp.net', 'HSJHJWH7323HSJSJ') // implement this on your end
await sock.sendMessage('1234@s.whatsapp.net', { forward: msg }) // WA forward the message!
```

## Reading Messages

A set of message keys must be explicitly marked read now. 
In multi-device, you cannot mark an entire "chat" read as it were with Baileys Web.
This means you have to keep track of unread messages.

``` ts
const key = {
    remoteJid: '1234-123@g.us',
    id: 'AHASHH123123AHGA', // id of the message you want to read
    participant: '912121232@s.whatsapp.net' // the ID of the user that sent the  message (undefined for individual chats)
}
// pass to readMessages function
// can pass multiple keys to read multiple messages as well
await sock.readMessages([key])
```

The message ID is the unique identifier of the message that you are marking as read. 
On a `WAMessage`, the `messageID` can be accessed using ```messageID = message.key.id```.

## Update Presence

``` ts
await sock.sendPresenceUpdate('available', id) 

```
This lets the person/group with ``` id ``` know whether you're online, offline, typing etc. 

``` presence ``` can be one of the following:
``` ts
type WAPresence = 'unavailable' | 'available' | 'composing' | 'recording' | 'paused'
```

The presence expires after about 10 seconds.

**Note:** In the multi-device version of WhatsApp -- if a desktop client is active, WA doesn't send push notifications to the device. If you would like to receive said notifications -- mark your Baileys client offline using `sock.sendPresenceUpdate('unavailable')`


## Downloading Media Messages

If you want to save the media you received
``` ts
import { writeFile } from 'fs/promises'
import { downloadMediaMessage } from 'baileys-erlangga'

sock.ev.on('messages.upsert', async ({ messages }) => {
    const m = messages[0]

    if (!m.message) return // if there is no text or media message
    const messageType = Object.keys (m.message)[0]// get what type of message it is -- text, image, video
    // if the message is an image
    if (messageType === 'imageMessage') {
        // download the message
        const buffer = await downloadMediaMessage(
            m,
            'buffer',
            { },
            { 
                logger,
                // pass this so that baileys can request a reupload of media
                // that has been deleted
                reuploadRequest: sock.updateMediaMessage
            }
        )
        // save to file
        await writeFile('./my-download.jpeg', buffer)
    }
}
```

**Note:** WhatsApp automatically removes old media from their servers. For the device to access said media -- a re-upload is required by another device that has it. This can be accomplished using: 
``` ts
const updatedMediaMsg = await sock.updateMediaMessage(msg)
```

## Deleting Messages

``` ts
const jid = '1234@s.whatsapp.net' // can also be a group
const response = await sock.sendMessage(jid, { text: 'hello!' }) // send a message
// sends a message to delete the given message
// this deletes the message for everyone
await sock.sendMessage(jid, { delete: response.key })
```

**Note:** deleting for oneself is supported via `chatModify` (next section)

## Updating Messages

``` ts
const jid = '1234@s.whatsapp.net'

await sock.sendMessage(jid, {
      text: 'updated text goes here',
      edit: response.key,
    });
```

## Modifying Chats

WA uses an encrypted form of communication to send chat/app updates. This has been implemented mostly and you can send the following updates:

- Archive a chat
  ``` ts
  const lastMsgInChat = await getLastMessageInChat('123456@s.whatsapp.net') // implement this on your end
  await sock.chatModify({ archive: true, lastMessages: [lastMsgInChat] }, '123456@s.whatsapp.net')
  ```
- Mute/unmute a chat
  ``` ts
  // mute for 8 hours
  await sock.chatModify({ mute: 8*60*60*1000 }, '123456@s.whatsapp.net', [])
  // unmute
  await sock.chatModify({ mute: null }, '123456@s.whatsapp.net', [])
  ```
- Mark a chat read/unread
  ``` ts
  const lastMsgInChat = await getLastMessageInChat('123456@s.whatsapp.net') // implement this on your end
  // mark it unread
  await sock.chatModify({ markRead: false, lastMessages: [lastMsgInChat] }, '123456@s.whatsapp.net')
  ```

- Delete a message for me
  ``` ts
  await sock.chatModify(
    { clear: { messages: [{ id: 'ATWYHDNNWU81732J', fromMe: true, timestamp: "1654823909" }] } }, 
    '123456@s.whatsapp.net', 
    []
    )

  ```

- Delete a chat
  ``` ts
  const lastMsgInChat = await getLastMessageInChat('123456@s.whatsapp.net') // implement this on your end
  await sock.chatModify({
    delete: true,
    lastMessages: [{ key: lastMsgInChat.key, messageTimestamp: lastMsgInChat.messageTimestamp }]
  },
  '123456@s.whatsapp.net')
  ```

- Pin/unpin a chat
  ``` ts
  await sock.chatModify({
    pin: true // or `false` to unpin
  },
  '123456@s.whatsapp.net')
  ```
  
- Star/unstar a message
  ``` ts
  await sock.chatModify({
  star: {
  	messages: [{ id: 'messageID', fromMe: true // or `false` }],
      	star: true // - true: Star Message; false: Unstar Message
  }},'123456@s.whatsapp.net');
  ```

**Note:** if you mess up one of your updates, WA can log you out of all your devices and you'll have to log in again.

## Disappearing Messages

``` ts
const jid = '1234@s.whatsapp.net' // can also be a group
// turn on disappearing messages
await sock.sendMessage(
    jid, 
    // this is 1 week in seconds -- how long you want messages to appear for
    { disappearingMessagesInChat: WA_DEFAULT_EPHEMERAL }
)
// will send as a disappearing message
await sock.sendMessage(jid, { text: 'hello' }, { ephemeralExpiration: WA_DEFAULT_EPHEMERAL })
// turn off disappearing messages
await sock.sendMessage(
    jid, 
    { disappearingMessagesInChat: false }
)

```

## Misc

- To check if a given ID is on WhatsApp
    ``` ts
    const id = '123456'
    const [result] = await sock.onWhatsApp(id)
    if (result.exists) console.log (`${id} exists on WhatsApp, as jid: ${result.jid}`)
    ```
- To query chat history on a group or with someone
    TODO, if possible
- To get the status of some person
    ``` ts
    const status = await sock.fetchStatus("xyz@s.whatsapp.net")
    console.log("status: " + status)
    ```
- To change your profile status
    ``` ts
    const status = 'Hello World!'
    await sock.updateProfileStatus(status)
    ```
- To change your profile name
    ``` ts
    const name = 'My name'
    await sock.updateProfileName(name)
    ```
- To get the display picture of some person/group
    ``` ts
    // for low res picture
    const ppUrl = await sock.profilePictureUrl("xyz@g.us")
    console.log("download profile picture from: " + ppUrl)
    // for high res picture
    const ppUrl = await sock.profilePictureUrl("xyz@g.us", 'image')
    ```
- To change your display picture or a group's
    ``` ts
    const jid = '111234567890-1594482450@g.us' // can be your own too
    await sock.updateProfilePicture(jid, { url: './new-profile-picture.jpeg' })
    ```
- To remove your display picture or a group's
    ``` ts
    const jid = '111234567890-1594482450@g.us' // can be your own too
    await sock.removeProfilePicture(jid)
    ```
- To get someone's presence (if they're typing or online)
    ``` ts
    // the presence update is fetched and called here
    sock.ev.on('presence.update', json => console.log(json))
    // request updates for a chat
    await sock.presenceSubscribe("xyz@s.whatsapp.net") 
    ```
- To block or unblock user
    ``` ts
    await sock.updateBlockStatus("xyz@s.whatsapp.net", "block") // Block user
    await sock.updateBlockStatus("xyz@s.whatsapp.net", "unblock") // Unblock user
    ```
- To get a business profile, such as description or category
    ```ts
    const profile = await sock.getBusinessProfile("xyz@s.whatsapp.net")
    console.log("business description: " + profile.description + ", category: " + profile.category)
    ```
Of course, replace ``` xyz ``` with an actual ID. 

## Groups
- To create a group
    ``` ts
    // title & participants
    const group = await sock.groupCreate("My Fab Group", ["1234@s.whatsapp.net", "4564@s.whatsapp.net"])
    console.log ("created group with id: " + group.gid)
    sock.sendMessage(group.id, { text: 'hello there' }) // say hello to everyone on the group
    ```
- To add/remove people to a group or demote/promote people
    ``` ts
    // id & people to add to the group (will throw error if it fails)
    const response = await sock.groupParticipantsUpdate(
        "abcd-xyz@g.us", 
        ["abcd@s.whatsapp.net", "efgh@s.whatsapp.net"],
        "add" // replace this parameter with "remove", "demote" or "promote"
    )
    ```
- To change the group's subject
    ``` ts
    await sock.groupUpdateSubject("abcd-xyz@g.us", "New Subject!")
    ```
- To change the group's description
    ``` ts
    await sock.groupUpdateDescription("abcd-xyz@g.us", "New Description!")
    ```
- To change group settings
    ``` ts
    // only allow admins to send messages
    await sock.groupSettingUpdate("abcd-xyz@g.us", 'announcement')
    // allow everyone to send messages
    await sock.groupSettingUpdate("abcd-xyz@g.us", 'not_announcement')
    // allow everyone to modify the group's settings -- like display picture etc.
    await sock.groupSettingUpdate("abcd-xyz@g.us", 'unlocked')
    // only allow admins to modify the group's settings
    await sock.groupSettingUpdate("abcd-xyz@g.us", 'locked')
    ```
- To leave a group
    ``` ts
    await sock.groupLeave("abcd-xyz@g.us") // (will throw error if it fails)
    ```
- To get the invite code for a group
    ``` ts
    const code = await sock.groupInviteCode("abcd-xyz@g.us")
    console.log("group code: " + code)
    ```
- To revoke the invite code in a group
    ```ts
    const code = await sock.groupRevokeInvite("abcd-xyz@g.us")
    console.log("New group code: " + code)
    ```
- To query the metadata of a group
    ``` ts
    const metadata = await sock.groupMetadata("abcd-xyz@g.us") 
    console.log(metadata.id + ", title: " + metadata.subject + ", description: " + metadata.desc)
    ```
- To join the group using the invitation code
    ``` ts
    const response = await sock.groupAcceptInvite("xxx")
    console.log("joined to: " + response)
    ```
    Of course, replace ``` xxx ``` with invitation code.
- To get group info by invite code
    ```ts
    const response = await sock.groupGetInviteInfo("xxx")
    console.log("group information: " + response)
    ```
- To join the group using groupInviteMessage
    ``` ts
    const response = await sock.groupAcceptInviteV4("abcd@s.whatsapp.net", groupInviteMessage)
    console.log("joined to: " + response)
    ```
  Of course, replace ``` xxx ``` with invitation code.

- To get list request join
    ``` ts
    const response = await sock.groupRequestParticipantsList("abcd-xyz@g.us")
    console.log(response)
    ```
- To approve/reject request join
    ``` ts
    const response = await sock.groupRequestParticipantsUpdate(
        "abcd-xyz@g.us", // id group,
        ["abcd@s.whatsapp.net", "efgh@s.whatsapp.net"],
        "approve" // replace this parameter with "reject" 
    )
    console.log(response)
    ```

## Channel
- To get newsletter info from code
    ```ts
    // https://whatsapp.com/channel/key
    const key = '123wedss972279'
    const result = await sock.getNewsletterInfo(key)
    console.log(result)
    ```
- To create newsletter
    ```ts
    const result = await sock.createNewsLetter('Name newsletter', 'Description news letter', { url: 'url profile pictur' })
    console.log(result)
    ```
- To get subscribed newsletters
    ```ts
    const result = await sock.getSubscribedNewsletters()
    console.log(result)
    ```
- To toggle mute newsletters
    ```ts
    const result = await sock.toggleMuteNewsletter(jid, true) // true to mute, false to unmute
    console.log(result)
    ```
- To follow newsletter
    ```ts
    const result = await sock.followNewsletter(jid)
    console.log(result)
    ```
- To unfollow newsletter
    ```ts
    const result = await sock.unFollowNewsletter(jid)
    console.log(result)
    ```


## Broadcast Lists & Stories

Messages can be sent to broadcasts & stories. 
you need to add the following message options in sendMessage, like this:
```ts
sock.sendMessage(jid, {image: {url: url}, caption: caption}, {backgroundColor : backgroundColor, font : font, statusJidList: statusJidList, broadcast : true})
```
- the message body can be a extendedTextMessage or imageMessage or videoMessage or voiceMessage
- You can add backgroundColor and other options in the message options
- broadcast: true enables broadcast mode
- statusJidList: a list of people that you can get which you need to provide, which are the people who will get this status message.

- You can send messages to broadcast lists the same way you send messages to groups & individual chats.
- Right now, WA Web does not support creating broadcast lists, but you can still delete them.
- Broadcast IDs are in the format `12345678@broadcast`
- To query a broadcast list's recipients & name:
    ``` ts
    const bList = await sock.getBroadcastListInfo("1234@broadcast")
    console.log (`list name: ${bList.name}, recps: ${bList.recipients}`)
    ```

## Writing Custom Functionality
Baileys is written with custom functionality in mind. Instead of forking the project & re-writing the internals, you can simply write your own extensions.

First, enable the logging of unhandled messages from WhatsApp by setting:
``` ts
const sock = makeWASocket({
    logger: P({ level: 'debug' }),
})
```
This will enable you to see all sorts of messages WhatsApp sends in the console. 

Some examples:

1. Functionality to track the battery percentage of your phone.
    You enable logging and you'll see a message about your battery pop up in the console: 
    ```{"level":10,"fromMe":false,"frame":{"tag":"ib","attrs":{"from":"@s.whatsapp.net"},"content":[{"tag":"edge_routing","attrs":{},"content":[{"tag":"routing_info","attrs":{},"content":{"type":"Buffer","data":[8,2,8,5]}}]}]},"msg":"communication"} ``` 
    
   The "frame" is what the message received is, it has three components:
   - `tag` -- what this frame is about (eg. message will have "message")
   - `attrs` -- a string key-value pair with some metadata (contains ID of the message usually)
   - `content` -- the actual data (eg. a message node will have the actual message content in it)
   - read more about this format [here](/src/WABinary/readme.md)

    You can register a callback for an event using the following:
    ``` ts
    // for any message with tag 'edge_routing'
    sock.ws.on(`CB:edge_routing`, (node: BinaryNode) => { })
    // for any message with tag 'edge_routing' and id attribute = abcd
    sock.ws.on(`CB:edge_routing,id:abcd`, (node: BinaryNode) => { })
    // for any message with tag 'edge_routing', id attribute = abcd & first content node routing_info
    sock.ws.on(`CB:edge_routing,id:abcd,routing_info`, (node: BinaryNode) => { })
    ```
 Also, this repo is now licenced under GPL 3 since it uses [libsignal-node](https://git.questbook.io/backend/service-coderunner/-/merge_requests/1)
