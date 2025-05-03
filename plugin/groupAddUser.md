<title>Group Add User</title>
<desc>Fitur ini memungkinkan Anda untuk menambahkan anggota baru ke dalam grup. Anda dapat mengundang orang lain untuk bergabung dengan grup</desc>
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
- Kirim pesan dengan format: `.add <628xxxxxxxxxx>`

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
	
	if (msg.toLowerCase().startsWith('.add')) {
        if (!remoteJid.endsWith('@g.us')) {
            return sock.sendMessage(remoteJid, { text: 'Perintah hanya bisa digunakan di grup.' }, { quoted: message });
        }

        const metadata = await sock.groupMetadata(remoteJid);
        const botNumber = sock.user.id.split(':')[0] + '@s.whatsapp.net';
        const botIsAdmin = metadata.participants.find(p => p.id === botNumber)?.admin !== null;

        if (!botIsAdmin) {
            return sock.sendMessage(remoteJid, { text: 'Bot harus menjadi admin untuk menggunakan perintah ini.' }, { quoted: message });
        }

        const number = msg.split('.add')[1]?.replace(/\D/g, '');
        if (!number) {
            return sock.sendMessage(remoteJid, {
                text: '⚠Masukkan nomor yang ingin ditambahkan.\nContoh: `.add 6281234567890`'
            }, { quoted: message });
        }

        const jid = number + '@s.whatsapp.net';

        try {
            await sock.groupParticipantsUpdate(remoteJid, [jid], 'add');
            await sock.sendMessage(remoteJid, {
                text: `Telah mengundang @${number} ke grup.`,
                mentions: [jid]
            }, { quoted: message });
        } catch (err) {
            console.error('Gagal:', err);
            await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
        }
    }
	
};

module.exports.SELF = true;
```