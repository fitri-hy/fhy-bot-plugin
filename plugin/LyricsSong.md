<title>Lirik Lagu</title>
<desc>Temukan lirik lagu terbaru, terpopuler, dan kenangan sepanjang masa di sini.</desc>
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
- Kirim pesan dengan format: `.lirik <judul-lagu>`

---

```js
const axios = require('axios');

const EMOJIS = {
    loading: '🕒',
    success: '✅',
    error: '❌'
};

module.exports = async (sock, message, msg, sender) => {
    if (msg && msg.toLowerCase().startsWith('.lirik')) {
        try {
            await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });

            const songTitle = msg.split('.lirik')[1]?.trim();
            if (!songTitle) {
                await sock.sendMessage(sender, { text: `Masukkan judul lagunya.` }, { quoted: message });
                await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
                return;
            }

            const apiUrl = `https://some-random-api.com/lyrics?title=${encodeURIComponent(songTitle)}`;
            const res = await axios.get(apiUrl);
            const { title, artist, lyrics, thumbnail } = res.data;

            if (!lyrics) {
                await sock.sendMessage(sender, { text: `Lirik tidak ditemukan.` }, { quoted: message });
                await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
                return;
            }

            const caption = `*${artist} - ${title}*\n\n${lyrics.substring(0, 2000)}${lyrics.length > 2000 ? '...' : ''}`;

            if (thumbnail) {
                const response = await axios.get(thumbnail, { responseType: 'arraybuffer' });
                const imageBuffer = Buffer.from(response.data, 'binary');
                await sock.sendMessage(sender, { image: imageBuffer, caption: caption }, { quoted: message });
            } else {
                await sock.sendMessage(sender, { text: caption }, { quoted: message });
            }

            await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });

        } catch (error) {
            console.error('Error:', error);
            if (error.response && error.response.status === 404) {
                await sock.sendMessage(sender, { text: `Lirik untuk lagu tersebut tidak ditemukan.` }, { quoted: message });
            } else {
                await sock.sendMessage(sender, { text: `Terjadi kesalahan saat mengambil lirik.` }, { quoted: message });
            }

            await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
        }
    }
};
```