<title>Terabox Downloader</title>
<desc>Terabox Downloader memungkinkan Anda untuk mengunduh video dengan mudah dan cepat dari platform Terabox.</desc>
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
- Kirim pesan teks ke bot dengan format: `..tdl <terabox-video-url>`

---

```
const axios = require("axios");
const { terabox } = require("nayan-videos-downloader");

const EMOJIS = {
    loading: '🕒',
    success: '✅',
    error: '❌'
};

module.exports = async (sock, message, msg, sender) => {
    if (msg && msg.startsWith('.tdl ')) {
        const url = msg.split('.tdl ')[1].trim();
        if (!url) return await sock.sendMessage(sender, { text: 'URL tidak valid!' }, { quoted: message });

        try {
			await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });
            const result = await terabox(url);
            const videoUrl = result?.data?.video || result?.data?.video2;

            if (!videoUrl) {
                throw new Error("Gagal mendapatkan URL video dari Terabox.");
            }

            const response = await axios.get(videoUrl, { responseType: 'arraybuffer' });
            const videoBuffer = Buffer.from(response.data, 'binary');

            await sock.sendMessage(sender, {
                video: videoBuffer,
                mimetype: 'video/mp4',
                caption: `Berikut video dari Terabox:\n${result.data.file_name || 'Tanpa Judul'}`
            }, { quoted: message });

            await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });

        } catch (err) {
            console.error("Terjadi kesalahan:", err);
            await sock.sendMessage(sender, { text: `${EMOJIS.error} Gagal mengunduh video. Coba lagi atau cek URL-nya.` }, { quoted: message });
            await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
        }
    }
};
```