<title>Lynix-AI</title>
<desc>Menggunakan kecerdasan buatan untuk memberikan respons percakapan yang alami dan akurat.</desc>
<github>fitri-hy</github>
<support>
  {
    "windows": "supported",
    "linux": "supported",
    "termux": "supported"
  }
</support>

## Instalasi:
Langkah-langkah instalasi pemasangan plugin Anda bisa lihat di [Dokumentasi](/docs#Plugin)

## Penggunaan:
- Kirim pesan dengan mention ke bot dengan format: `@<mention-bot> <text>` & untuk reset history `!clear`
- Kirim reply ke pesan bot dengan format: `<text>` & untuk reset history `!clear`
- Dapatkan ApiKey [*DISINI*](https://lynix.i-as.dev/docs#limits)

---

```
const axios = require('axios');

const EMOJIS = {
    loading: '🕒',
    success: '✅',
    error: '❌'
};

module.exports = async (sock, message, msg, sender) => {
    const contextInfo = message.message?.extendedTextMessage?.contextInfo;
    const botJid = sock.user.id.split(':')[0] + "@s.whatsapp.net";
    const quotedMessage = contextInfo?.quotedMessage;
    const mentionedJid = contextInfo?.mentionedJid || [];
    const isBotMentioned = mentionedJid.includes(botJid);
    const quotedSender = contextInfo?.participant;

    if (!isBotMentioned && quotedSender !== botJid) {
        return;
    }

    const regex = new RegExp(`@${botJid.split('@')[0]}`, 'g');
    const question = msg.replace(regex, '').trim();

    if (question.length === 0) {
        return;
    }

    if (question.toLowerCase().includes('!clear')) {
        try {
            await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });

            const response = await axios.post('https://lynix.i-as.dev/api/clear', {
                user_id: sender
            });

            if (response.data.success) {
                await sock.sendMessage(sender, { text: "Memori percakapan berhasil dihapus!", quoted: message });
            } else {
                await sock.sendMessage(sender, { text: "Terjadi kesalahan saat menghapus memori percakapan.", quoted: message });
            }

            await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });
        } catch (err) {
            console.error('Error clear memory API:', err.message);
            await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
        }
        return;
    }

    try {
        await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });

        const response = await axios.post('https://lynix.i-as.dev/api/chat', {
            user_id: sender,
            question: question,
            prompt: "Kamu adalah Lynix-AI, balas setia pertanyaan dengan sopan namun singkat."
        }, {
            headers: {
                'x-api-key': 'lynix-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx',
                'Content-Type': 'application/json'
            }
        });

        const aiAnswer = response.data?.answer || "Gak mengerti.";
        await sock.sendMessage(sender, { text: aiAnswer, }, { quoted: message });
        await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });

    } catch (err) {
        console.error('Error AI API:', err.message);
        await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
    }
};

module.exports.SELF = false;
```