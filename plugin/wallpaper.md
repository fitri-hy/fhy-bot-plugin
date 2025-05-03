<title>Wallpaper</title>
<desc>Wallpaper cantik dan berkualitas tinggi untuk desktop atau perangkat seluler Anda.</desc>
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
- Kirim pesan dengan format: `.wallpaper <teks>`

---

```js
const axios = require('axios');
const cheerio = require('cheerio');

const EMOJIS = {
    loading: '🕒',
    success: '✅',
    error: '❌'
};

module.exports = async (sock, message, msg, sender) => {
    if (msg && msg.toLowerCase().startsWith('.wallpaper')) {
        const query = msg.split(' ').slice(1).join(' ').trim();
        if (!query) {
            return sock.sendMessage(sender, { text: 'Masukkan kata kunci pencarian! Contoh: *.walpaper [teks]*' }, { quoted: message });
        }

        const page = 1;
        const url = `https://www.besthdwallpaper.com/search?CurrentPage=${page}&q=${query}`;

        try {
            const { data } = await axios.get(url);
            const $ = cheerio.load(data);
            const results = [];

            $('div.grid-item').each((index, element) => {
                const image = [
                    $(element).find('picture > img').attr('data-src') || $(element).find('picture > img').attr('src'),
                    $(element).find('picture > source:nth-child(1)').attr('srcset'),
                    $(element).find('picture > source:nth-child(2)').attr('srcset')
                ].filter(Boolean);

                if (image.length > 0) {
                    results.push({ image });
                }
            });

            if (results.length === 0) {
                return sock.sendMessage(sender, { text: `Tidak ditemukan wallpaper untuk kata kunci *${query}*` }, { quoted: message });
            }

            const bestWallpaper = results[0];
            const imageUrl = bestWallpaper.image[0];

            await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });

            const response = await axios.get(imageUrl, { responseType: 'arraybuffer' });
            const imageBuffer = Buffer.from(response.data, 'binary');

            const caption = 'Berhasil mendapatkan wallpaper';
            await sock.sendMessage(sender, {
                image: imageBuffer,
                caption: caption
            }, { quoted: message });

            await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });

        } catch (error) {
            console.error('GAGAL', error.message);
            await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
        }
    }
};
```