<title>Group Description Change</title>
<desc>Dengan fitur ini, Anda dapat mengubah deskripsi grup untuk memberikan informasi yang lebih jelas atau mengupdate tujuan grup.</desc>
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
- Kirim pesan dengan format: `.group-desc <text>`

---

```js
const EMOJIS = {
    loading: '🕒',
    success: '✅',
    error: '❌'
};

module.exports = async (sock, message, msg, sender) => {
    const remoteJid = message.key?.remoteJid;

    if (!remoteJid || !remoteJid.endsWith('@g.us')) return;

    if (typeof msg !== 'string') return;
	
	if (msg.toLowerCase().startsWith('.group-desc')) {
        const newDesc = msg.split('.group-desc')[1]?.trim();

        if (!newDesc) {
            return sock.sendMessage(remoteJid, {
                text: 'Masukkan deskripsi grup baru.\nContoh: `.group-desc Selamat datang di grup kita!`'
            }, { quoted: message });
        }

        try {
            await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });
            await sock.groupUpdateDescription(remoteJid, newDesc);
            await sock.sendMessage(remoteJid, {
                text: `Deskripsi grup berhasil diubah menjadi:\n*${newDesc}*`
            }, { quoted: message });
            await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });
        } catch (err) {
            console.error('Gagal:', err);
            await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
        }
    }
	
};

module.exports.SELF = true;
```