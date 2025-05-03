<title>Quote Generator</title>
<desc>Buat gambar quote estetis dengan teks dan nama pengguna secara otomatis. Cocok untuk status, story, dan inspirasi harian.</desc>
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
- Kirim pesan teks ke bot dengan format: `.quote <teks>`

---

```
const Jimp = require('jimp');
const axios = require('axios');

const EMOJIS = {
    loading: '🕒',
    success: '✅',
    error: '❌'
};

const QUOTE_IMAGE_URL = 'https://static.vecteezy.com/system/resources/thumbnails/052/248/075/small_2x/peacock-feather-wallpaper-hd-wallpaper-photo.jpeg';

module.exports = async (sock, message, msg, sender) => {
    try {
        if (!msg || !msg.toLowerCase().startsWith('.quote')) return;

        const quoteTextRaw = msg.substring(6).trim();
        if (!quoteTextRaw) {
            return sock.sendMessage(sender, { text: '❗ Format salah.\nGunakan: `.quote <text>`'}, { quoted: message });
        }

		await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });

        const response = await axios.get(QUOTE_IMAGE_URL, { responseType: 'arraybuffer' });
        const image = await Jimp.read(response.data);
        const fontQuote = await Jimp.loadFont(Jimp.FONT_SANS_32_WHITE);
        const fontUser = await Jimp.loadFont(Jimp.FONT_SANS_16_WHITE);
        const textWidth = image.bitmap.width - 40;
        const username = message.pushName || sender.split('@')[0];
        const profilePicUrl = await sock.profilePictureUrl(sender, 'image');
        const profilePicBuffer = await axios.get(profilePicUrl, { responseType: 'arraybuffer' });
        const profilePicImage = await Jimp.read(profilePicBuffer.data);
        profilePicImage.resize(100, 100).circle();
        const quoteText = `"${quoteTextRaw}"`;
        const quoteHeight = Jimp.measureTextHeight(fontQuote, quoteText, textWidth);
        const userText = `@${username}`;
        const usernameHeight = Jimp.measureTextHeight(fontUser, userText, textWidth);
        const profilePicHeight = 100;

        const spacing = {
            topPadding: 40,
            betweenUsernameAndQuote: 30,
            betweenProfileAndUsername: 20
        };

        const totalHeight = spacing.topPadding + profilePicHeight + spacing.betweenProfileAndUsername + usernameHeight + spacing.betweenUsernameAndQuote + quoteHeight;
        const startY = (image.bitmap.height - totalHeight) / 2;
        const overlay = new Jimp(image.bitmap.width, image.bitmap.height, '#00000099');
        image.composite(overlay, 0, 0);

        const profileX = (image.bitmap.width - 100) / 2;
        const profileY = startY;
        const usernameY = profileY + profilePicHeight + spacing.betweenProfileAndUsername;
        const quoteY = usernameY + usernameHeight + spacing.betweenUsernameAndQuote;

        image.composite(profilePicImage, profileX, profileY);
        image.print(fontUser, 20, usernameY, {
            text: userText,
            alignmentX: Jimp.HORIZONTAL_ALIGN_CENTER,
            alignmentY: Jimp.VERTICAL_ALIGN_TOP
        }, textWidth, usernameHeight);
        image.print(fontQuote, 20, quoteY, {
            text: quoteText,
            alignmentX: Jimp.HORIZONTAL_ALIGN_CENTER,
            alignmentY: Jimp.VERTICAL_ALIGN_TOP
        }, textWidth, quoteHeight);
		
        const buffer = await image.getBufferAsync(Jimp.MIME_JPEG);
        await sock.sendMessage(sender, {
            image: buffer,
            caption: '📝 Quote Image',
        }, { quoted: message });

		await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });

    } catch (err) {
        console.error('Error generating quote image:', err);
		await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
    }
};
```