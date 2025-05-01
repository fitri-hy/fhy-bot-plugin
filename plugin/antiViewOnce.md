<title>Anti View Once</title>
<desc>Memungkinkan pengguna untuk melihat kembali pesan media (foto atau video) yang dikirim dengan mode sekali lihat.</desc>
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
- Kirim pesan dengan kutip pesan sekali lihat ke bot dengan format: `.anti-viewonce`

---

```js
const { downloadMediaMessage, getContentType } = require('@whiskeysockets/baileys');

const EMOJIS = {
    loading: '🕒',
    success: '✅',
    error: '❌'
};

module.exports = async (sock, message, msg, sender) => {
    if (msg && msg.toLowerCase() === '.anti-viewonce') {
        const context = message.message?.extendedTextMessage?.contextInfo;

        if (!context || !context.quotedMessage) {
            await sock.sendMessage(sender, {
                text: `Tidak ada pesan sekali lihat yang di-quote.`,
            }, { quoted: message });
            return;
        }

        const quotedMessage = context.quotedMessage;
        const isViewOnce = quotedMessage.viewOnceMessage || Object.values(quotedMessage).some(m => m?.viewOnce);

        if (!isViewOnce) {
            await sock.sendMessage(sender, {
                text: `Pesan yang di-quote bukan pesan sekali lihat.`,
            }, { quoted: message });
            return;
        }

        let viewOnceMsg = quotedMessage.viewOnceMessage?.message || quotedMessage;
        let viewOnceType = getContentType(viewOnceMsg);
        let mediaMessage = viewOnceMsg[viewOnceType];

        if (mediaMessage?.viewOnce) {
            delete mediaMessage.viewOnce;
        }

		await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });

        try {
            const buffer = await downloadMediaMessage(
                { message: { [viewOnceType]: mediaMessage } },
                'buffer',
                {},
                { reuploadRequest: sock.updateMediaMessage }
            );

            if (viewOnceType === 'imageMessage') {
                await sock.sendMessage(sender, {
                    image: buffer,
                    caption: 'Anti Sekali Lihat Berhasil (Gambar)',
                }, { quoted: message });
            } else if (viewOnceType === 'videoMessage') {
                await sock.sendMessage(sender, {
                    video: buffer,
                    caption: 'Anti Sekali Lihat Berhasil (Video)',
                }, { quoted: message });
            } else {
                await sock.sendMessage(sender, {
                    text: `Jenis pesan tidak didukung.`,
                }, { quoted: message });
            }

			await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });

        } catch (err) {
            console.error('Gagal mengunduh media:', err);
			await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
        }
    }
};
```