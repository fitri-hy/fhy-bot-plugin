<title>Group Demote User</title>
<desc>Fitur ini memungkinkan Anda untuk menurunkan peran atau status seorang anggota dalam grup, misalnya mengubah admin menjadi anggota biasa.</desc>
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
- Kirim pesan dengan format: `.demote <mention>` atau kutip pesan `.demote`

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
	
	if (msg.toLowerCase().startsWith('.demote')) {
        if (!remoteJid.endsWith('@g.us')) {
            return sock.sendMessage(remoteJid, { text: '❌ Perintah hanya bisa digunakan di grup.' }, { quoted: message });
        }

        const metadata = await sock.groupMetadata(remoteJid);
        const botNumber = sock.user.id.split(':')[0] + '@s.whatsapp.net';
        const botIsAdmin = metadata.participants.find(p => p.id === botNumber)?.admin !== null;

        if (!botIsAdmin) {
            return sock.sendMessage(remoteJid, { text: 'Bot harus menjadi admin untuk menggunakan perintah ini.' }, { quoted: message });
        }

        let targetId;

        const mentionedJid = message.message?.extendedTextMessage?.contextInfo?.mentionedJid;
        if (mentionedJid && mentionedJid.length > 0) {
            targetId = mentionedJid[0];
        }

        const quoted = message.message?.extendedTextMessage?.contextInfo;
        if (!targetId && quoted?.participant) {
            targetId = quoted.participant;
        }

        if (!targetId) {
            return sock.sendMessage(remoteJid, {
                text: 'Sebut pengguna atau balas pesan mereka untuk didemote.\nContoh: `.demote @user` atau balas pesan dengan `.demote`'
            }, { quoted: message });
        }

        try {
            await sock.groupParticipantsUpdate(remoteJid, [targetId], 'demote');
            await sock.sendMessage(remoteJid, {
                text: `Pengguna @${targetId.split('@')[0]} telah didemote menjadi member.`,
                mentions: [targetId]
            }, { quoted: message });
        } catch (err) {
            console.error('Gagal:', err);
            await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
        }
    }
	
};

module.exports.SELF = true;
```