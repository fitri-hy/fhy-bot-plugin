<title>Terjemahan</title>
<desc>Menerjemahkan bahasa ke bahasa lain sesuai pilihan</desc>
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
- Kirim pesan teks ke bot dengan format: `.translate <kode_bahasa> <teks>`

---

```
const axios = require('axios');

const EMOJIS = {
    loading: '🕒',
    success: '✅',
    error: '❌'
};

module.exports = async (sock, message, msg, sender) => {
    if (msg && msg.toLowerCase().startsWith('.translate')) {
        const args = msg.split(' ').slice(1);

        if (args.length < 2) {
            await sock.sendMessage(sender, { text: 'Contoh: .translate id Hello, how are you?' }, { quoted: message });
            return;
        }

        const targetLang = args[0];
        const text = args.slice(1).join(' ');

        try {
			await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });
            const url = `https://translate.googleapis.com/translate_a/single?client=gtx&sl=auto&tl=${targetLang}&dt=t&q=${encodeURIComponent(text)}`;
            const { data } = await axios.get(url);

            const translatedText = data[0].map(item => item[0]).join('');
            const detectedLang = data[2];

            const responseText = `🌐 *Terjemahan* (${detectedLang} ➜ ${targetLang}):\n\n${translatedText}`.trim();

            await sock.sendMessage(sender, { text: responseText }, { quoted: message });
			await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });
        } catch (err) {
            console.error('Translate error:', err.message);
			await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
        }
    }
};
```