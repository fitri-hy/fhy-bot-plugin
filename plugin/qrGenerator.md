<title>QR Generator</title>
<desc>Alat untuk membuat kode QR yang dapat digunakan untuk berbagai tujuan, seperti berbagi informasi kontak, URL, teks, dan lainnya, dengan mudah dan cepat.</desc>
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
- Kirim pesan teks ke bot dengan format: `.qr <text>`

---

```js
const QRCode = require('qrcode');
const { MessageType } = require('@whiskeysockets/baileys');

const EMOJIS = {
    loading: '🕒',
    success: '✅',
    error: '❌'
};

module.exports = async (sock, message, msg, sender) => {
    if (msg && msg.toLowerCase().startsWith('.qr')) {
        const text = msg.slice(3).trim();
        if (!text) {
            return sock.sendMessage(sender, { text: '❗ Masukkan teks untuk dijadikan QR code.\nContoh: `.qr Halo Dunia`' }, { quoted: message });
        }

        try {
			await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });
            const qrDataUrl = await QRCode.toDataURL(text);
            const qrImageBuffer = Buffer.from(qrDataUrl.split(',')[1], 'base64');

            await sock.sendMessage(sender, {
                image: qrImageBuffer,
                caption: `*QR Berhasil* \n> ${text}`
            }, { quoted: message });

			await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });
        } catch (err) {
			await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
        }
    }
};
```