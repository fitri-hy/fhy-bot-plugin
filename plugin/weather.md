<title>Cuaca</title>
<desc>Perkiraan cuaca di berbagai kota</desc>
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
- Kirim pesan teks ke bot dengan format: `.weather <kota>`

---

```
const axios = require('axios');

const EMOJIS = {
    loading: '🕒',
    success: '✅',
    error: '❌'
};

module.exports = async (sock, message, msg, sender) => {
    if (msg && msg.toLowerCase().startsWith('.weather')) {
        const cityName = msg.split(' ').slice(1).join(' ').trim();

        if (!cityName) {
            await sock.sendMessage(sender, { text: 'Contoh: .weather Jakarta' }, { quoted: message });
            return;
        }

        try {
			await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });
            const weatherUrl = `https://wttr.in/${cityName}?format=%t|%C|%w|%h&lang=id&m`;
            const response = await axios.get(weatherUrl);
            const weatherData = response.data;

            const weatherParts = weatherData.split('|');
            const temperature = weatherParts[0];
            const condition = weatherParts[1];
            const wind = weatherParts[2];
            const humidity = weatherParts[3];

            const weatherJson = {
                temperature: temperature.trim(),
                condition: condition.trim(),
                wind: wind.trim(),
                humidity: humidity.trim()
            };

            const weatherText = `*Cuaca di ${cityName}:*\n\n🌡️ *Suhu:* ${weatherJson.temperature}\n🌤️ *Kondisi:* ${weatherJson.condition}\n🌬️ *Angin:* ${weatherJson.wind}\n💧 *Kelembapan:* ${weatherJson.humidity}`.trim();

            await sock.sendMessage(sender, { text: weatherText }, { quoted: message });
			await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });
        } catch (err) {
            console.error('Weather Error:', err.message);
			await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
        }
    }
};
```