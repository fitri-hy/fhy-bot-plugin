<title>Group Name Change</title>
<desc>Fitur ini memungkinkan Anda untuk mengubah nama grup. Anda dapat memperbarui nama grup sesuai dengan kebutuhan.</desc>
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
- Kirim pesan dengan format: `.group-name <text>`

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
	
	if (msg.toLowerCase().startsWith('.group-name')) {
        const newName = msg.split('.group-name')[1]?.trim();

        if (!newName) {
            return sock.sendMessage(remoteJid, {
                text: 'Masukkan nama grup baru.\nContoh: `.group-name Grup Baru`'
            }, { quoted: message });
        }

        try {
            await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });
            await sock.groupUpdateSubject(remoteJid, newName);
            await sock.sendMessage(remoteJid, {
                text: `Nama grup berhasil diubah menjadi:\n*${newName}*`
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