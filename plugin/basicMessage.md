<title>Auto Reply Default</title>
<desc>Fitur Auto Reply Default untuk membalas pesan secara otomatis.</desc>
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
- Kirim pesan teks ke bot dengan format: `.text`, `.img`, `.audio`, `.video`, `.location`

---

```js
const { Mimetype } = require('@whiskeysockets/baileys');
const axios = require('axios');

const EMOJIS = {
    loading: '🕒',
    success: '✅',
    error: '❌'
};

module.exports = async (sock, message, msg, sender) => {
	
	// Text
	if (msg && msg.toLowerCase() === '.text') {
		await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });
		await sock.sendMessage(sender, { text: 'Example of a reply message text' }, { quoted: message });
		await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });
	}
		
	// Image
	if (msg && msg.toLowerCase() === '.img') {
		await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });
		const imageUrl = 'https://static.vecteezy.com/system/resources/thumbnails/052/248/075/small_2x/peacock-feather-wallpaper-hd-wallpaper-photo.jpeg';
		const response = await axios.get(imageUrl, { responseType: 'arraybuffer' });
		const imageBuffer = Buffer.from(response.data, 'binary');
		await sock.sendMessage(sender, { image: imageBuffer, caption: 'Example of a reply message img' }, { quoted: message });
		await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });
	}
		
	// Audio
	if (msg && msg.toLowerCase() === '.audio') {
		await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });
		const audioUrl = 'https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3';
		const { data } = await axios.get(audioUrl, { responseType: 'arraybuffer' });
		await sock.sendMessage(sender, {
		audio: Buffer.from(data, 'binary'),
			mimetype: 'audio/mp4',
			ptt: false
		}, { quoted: message });
		await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });
	}
		
	// Video
	if (msg && msg.toLowerCase() === '.video') {
		await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });
		const videoUrl = 'https://www.sample-videos.com/video321/mp4/720/big_buck_bunny_720p_1mb.mp4';
		const { data } = await axios.get(videoUrl, { responseType: 'arraybuffer' });
		await sock.sendMessage(sender, {
			video: Buffer.from(data, 'binary'),
			mimetype: 'video/mp4',
			caption: 'Example of a reply message video'
		}, { quoted: message });
		await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });
	}
		
	// Location
	if (msg && msg.toLowerCase() === '.location') {
		await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });
		await sock.sendMessage(sender, {
			location: { degreesLatitude: -6.1751, degreesLongitude: 106.8650 },
		}, { quoted: message });
		await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });
	}
	
};
```