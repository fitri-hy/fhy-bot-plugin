<title>Youtube Download</title>
<desc>Mengunduh video Youtube melalui pesan secara mudah</desc>
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
- Kirim pesan teks ke bot dengan format: `.ytdl <url_video>`

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
    if (!msg.toLowerCase().startsWith('.ytdl')) return;
    const url = msg.split(' ')[1];

    if (!url) {
        await sock.sendMessage(sender, {
            text: 'Invalid YouTube URL. Example: `.ytdl https://youtu.be/xxx`'
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
            addHeader: ['referer:youtube.com', 'user-agent:googlebot']
        });

        if (!output || !output.title) {
            throw new Error('Unable to retrieve video information.');
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
                console.log(`File ${videoPath} telah dihapus.`);
            }
        });

        await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });

    } catch (error) {
        console.error('Error download YouTube:', error);
        await sock.sendMessage(sender, { text: 'Failed to download video.', }, { quoted: message });
        await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
    }
};
```