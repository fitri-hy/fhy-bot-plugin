<title>Text2Voice</title>
<desc>Ubah teks menjadi suara secara instan menggunakan perintah sederhana. Dukungan berbagai bahasa seperti Indonesia, Inggris, Jepang, dan banyak lagi.</desc>
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
- Kirim pesan teks ke bot dengan format: `.text2voice <lang-id> <text>`

---

```
const { MessageType } = require('@whiskeysockets/baileys');
const { exec } = require('child_process');
const fs = require('fs');
const path = require('path');
const gTTS = require('gtts');

const EMOJIS = {
    loading: '🕒️',
    success: '✅',
    error: '❌'
};

module.exports = async (sock, message, msg, sender) => {
    try {
        if (!msg || !msg.toLowerCase().startsWith('.text2voice')) return;

        const args = msg.split(' ');
        if (args.length < 3) {
            await sock.sendMessage(sender, {
                text: '❗ Format salah.\nGunakan: `.text2voice <lang-id> <text>`\nContoh: `.text2voice id Halo dunia`'
            }, { quoted: message });
            return;
        }

        const lang = args[1];
        const text = args.slice(2).join(' ');

		await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });

        const filename = `voice_${Date.now()}.mp3`;
        const filepath = path.join(__dirname, filename);

        const gtts = new gTTS(text, lang);
        gtts.save(filepath, async (err) => {
            if (err) {
                console.error('TTS Error:', err);
                await sock.sendMessage(sender, {
                    text: `${EMOJIS.error} Gagal membuat suara.`
                }, { quoted: message });
                return;
            }

            const audioBuffer = fs.readFileSync(filepath);

            await sock.sendMessage(sender, {
                audio: audioBuffer,
                mimetype: 'audio/mpeg',
                ptt: true
            }, { quoted: message });

            fs.unlinkSync(filepath);

			await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });
        });

    } catch (err) {
        console.error('Error generating voice:', err);
		await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
    }
};
```