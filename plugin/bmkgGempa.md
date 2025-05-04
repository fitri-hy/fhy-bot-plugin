<title>Gempa BMKG</title>
<desc>Informasi terkini mengenai gempa bumi yang tercatat oleh BMKG, termasuk lokasi, kekuatan, kedalaman, serta potensi dampaknya.</desc>
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
- Kirim pesan teks ke bot dengan format: `.bmkg-gempa`

---

```
const axios = require('axios');

const EMOJIS = {
    loading: '🕒',
    success: '✅',
    error: '❌'
};

module.exports = async (sock, message, msg, sender) => {
    if (!msg.startsWith('.bmkg-gempa')) return;

    try {
        await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });

        const res = await axios.get('https://data.bmkg.go.id/DataMKG/TEWS/gempaterkini.json');
        const dataGempa = res.data.Infogempa.gempa.slice(0, 5);

        let teks = '📍 *5 Info Gempa Terkini dari BMKG*\n\n';

        dataGempa.forEach((gempa, i) => {
            teks += `*${i + 1}. ${gempa.Wilayah}*\n` +
                `🕒 ${gempa.Tanggal} ${gempa.Jam}\n` +
                `💥 Magnitudo: ${gempa.Magnitude}\n` +
                `📍 Kedalaman: ${gempa.Kedalaman}\n` +
                `📌 Lokasi: ${gempa.Lintang}, ${gempa.Bujur}\n` +
                `⚠️ Potensi: ${gempa.Potensi}\n\n`;
        });

        await sock.sendMessage(sender, { text: teks.trim() }, { quoted: message });
        await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });

    } catch (error) {
        console.error("BMKG Error:", error.message);
        await sock.sendMessage(sender, {
            react: { text: EMOJIS.error, key: message.key }
        });
    }
};
```