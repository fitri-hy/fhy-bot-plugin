<title>X Trending</title>
<desc>Fitur ini memungkinkan bot memperoleh data tagar yang sedang tren di X berdasarkan negara.</desc>
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
- Kirim pesan dengan format: `.xtrending <country-name>`

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
    const remoteJid = message.key?.remoteJid;
    if (!remoteJid || typeof msg !== 'string') return;

    if (msg.toLowerCase().startsWith('.xtrending')) {
        await sock.sendMessage(remoteJid, { react: { text: EMOJIS.loading, key: message.key } });

        const args = msg.split(' ');
        const country = args[1]?.toLowerCase() || 'indonesia';
        const url = `https://getdaytrends.com/${country}/`;

        try {
            const response = await axios.get(url);
            const $ = cheerio.load(response.data);
            let trends = [];

            $('tr').each((index, element) => {
                const trend = $(element);
                const position = trend.find('th').text().trim();
                const name = trend.find('.main a').text().trim();
                const tweetCount = trend.find('.desc .small').text().trim();

                if (position && name) {
                    trends.push(`${position}. ${name}\n> ${tweetCount ? `(${tweetCount})` : ''}`);
                }
            });

            if (trends.length === 0) {
                return sock.sendMessage(remoteJid, { text: `Tidak ada tren ditemukan untuk *${country}*.` }, { quoted: message });
            }

            const trendingText = `📈 *Trending X - ${country.toUpperCase()}*\n\n${trends.slice(0, 50).join('\n')}`;

            await sock.sendMessage(remoteJid, { text: trendingText }, { quoted: message });
            await sock.sendMessage(remoteJid, { react: { text: EMOJIS.success, key: message.key } });
        } catch (error) {
            console.error('Gagal ambil data tren:', error.message);
            await sock.sendMessage(remoteJid, { react: { text: EMOJIS.error, key: message.key } });
        }
    }
};
```