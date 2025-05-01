<title>Instagram Download</title>
<desc>Mengunduh video Instagram melalui pesan secara mudah</desc>
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
- Kirim pesan teks ke bot dengan format:  `.igdl <url_video>`

---

```js
const fs = require('fs');
const path = require('path');
const youtubedl = require('youtube-dl-exec');
const { Mimetype } = require('@whiskeysockets/baileys');

const EMOJIS = {
    loading: '🕒',
    success: '✅',
    error: '❌'
};

module.exports = async (sock, message, msg, sender) => {
    if (!msg.toLowerCase().startsWith('.igdl')) return;

    const url = msg.split(' ')[1];

    if (!url) {
        await sock.sendMessage(sender, {
            text: 'Invalid Instagram URL. Example: `.igdl https://www.instagram.com/p/xxx`'
        }, { quoted: message });
        return;
    }

    await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });

    try {
        const output = await youtubedl(url, {
            dumpSingleJson: true,
            noCheckCertificates: true,
            noWarnings: true,
            preferFreeFormats: true,
            addHeader: ['referer:instagram.com', 'user-agent:googlebot']
        });

        if (!output || !output.title) {
            throw new Error('Unable to retrieve Instagram video data.');
        }

        const title = output.title;
        const videoUrl = output.url;
        const videoPath = path.join(__dirname, `${title.replace(/[^\w\s]/gi, '')}.mp4`);

        await youtubedl(url, {
            output: videoPath,
            noWarnings: true,
            quiet: true
        });

        await sock.sendMessage(sender, {
            video: { url: videoPath },
            mimetype: 'video/mp4',
            caption: `${title}`,
        }, { quoted: message });

        fs.unlink(videoPath, (err) => {
            if (err) {
                console.error('Failed to delete file:', err);
            } else {
                console.log(`File ${videoPath} has been deleted.`);
            }
        });

        await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });

    } catch (error) {
        console.error('Error downloading Instagram video:', error);
        await sock.sendMessage(sender, { text: 'Failed to download Instagram video. Please try again later.', }, { quoted: message });
        await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
    }
};
```