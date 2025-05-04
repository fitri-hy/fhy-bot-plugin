<title>Waktu Sholat</title>
<desc>Informasi mengenai jadwal waktu sholat harian berdasarkan kota tertentu di indonesia.</desc>
<github>fitri-hy</github>
<support>
  {
    "windows": "supported",
    "linux": "supported",
    "termux": "not_supported"
  }
</support>

## Instalasi:
Langkah-langkah instalasi pemasangan plugin Anda bisa lihat di [Dokumentasi](/docs#Plugin)

## Penggunaan:
- Kirim pesan teks ke bot dengan format: `.sholat-time <kota>`

---

```
const axios = require('axios');

const EMOJIS = {
    loading: '🕒',
    success: '✅',
    error: '❌'
};

module.exports = async (sock, message, msg, sender) => {
    if (msg && msg.toLowerCase().startsWith('.sholat-time ')) {
        const location = msg.split(' ')[1];

        if (!location) {
            await sock.sendMessage(sender, { text: 'Please provide a city name after the command, like `.sholat-time Jakarta`.' });
            return;
        }
		
        try {
			await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });
            const response = await axios.get(`http://api.aladhan.com/v1/timingsByCity`, {
                params: {
                    city: location,
                    country: 'Indonesia',
                    method: 2,
                }
            });

            const data = response.data.data.timings;
            const prayerTimesMessage = `⏰ Waktu Sholat di ${location}:\n\nFajr: ${data.Fajr}\nDhuhr: ${data.Dhuhr}\nAsr: ${data.Asr}\nMaghrib: ${data.Maghrib}\nIsha: ${data.Isha}`;
            await sock.sendMessage(sender, { text: prayerTimesMessage }, { quoted: message });
            await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });

        } catch (error) {
            console.error(error);
            await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
        }
    }
};
```