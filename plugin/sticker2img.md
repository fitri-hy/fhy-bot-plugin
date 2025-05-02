<title>Sticker To Image</title>
<desc>Membuat Gambar secara otomatis dari pesan sticker.</desc>
<github>fitri-hy</github>
<support>
  {
    "windows": "supported",
    "linux": "supported",
    "termux": "unknown"
  }
</support>

## Instalasi:
Langkah-langkah instalasi pemasangan plugin Anda bisa lihat di [Dokumentasi](/docs#Plugin)

## Penggunaan:
- Kirim pesan dengan kutip gambar ke bot dengan format: `.stickerify`

---

```js
const { downloadContentFromMessage } = require('@whiskeysockets/baileys');

const EMOJIS = {
    loading: '🕒',
    success: '✅',
    error: '❌'
};

module.exports = async (sock, message, msg, sender) => {
    if (!msg.toLowerCase().startsWith('.stickerify')) return;

    const quotedMsg = message.message?.extendedTextMessage?.contextInfo?.quotedMessage;
    const imageMsg = quotedMsg?.imageMessage;
	
    if (!imageMsg) {
        await sock.sendMessage(sender, { text: 'Please reply to the image with the command `.stickerify`.' }, { quoted: message });
        return;
    }
	
    await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });

    try {
        const stream = await downloadContentFromMessage(imageMsg, 'image');
        let buffer = Buffer.alloc(0);
        for await (const chunk of stream) {
            buffer = Buffer.concat([buffer, chunk]);
        }

        await sock.sendMessage(sender, { sticker: buffer, }, { quoted: message });
        await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });

    } catch (error) {
        console.error('Error creating stickers:', error);
        await sock.sendMessage(sender, { text: 'Failed to convert image to sticker.', }, { quoted: message });
        await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
    }
};
```