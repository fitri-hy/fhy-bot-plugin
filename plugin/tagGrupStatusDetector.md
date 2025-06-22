<title>Mendeteksi Mention Grup Status</title>
<desc>Fitur ini memungkinkan pengguna untuk mendeteksi apakah sebuah grup telah disebut (mention) dalam status pengguna lain</desc>
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
- Otomatis mendeteksi setiap kali ada Mention Grup Status.

---

```
module.exports = async (sock, message, msg, sender) => {
    const id = message.key.remoteJid;
    const participant = message.key.participant || sender;
    const username = participant.split('@')[0];

    const isOnlyMessageContextInfo = (
        message.message &&
        Object.keys(message.message).length === 1 &&
        message.message.messageContextInfo
    );

    if (!msg && isOnlyMessageContextInfo) {
        try {
            const metadata = await sock.groupMetadata(id);
            const admins = metadata.participants.filter(
                (p) => p.admin === 'admin' || p.admin === 'superadmin'
            );
            const adminMentions = admins.map((a) => a.id);
            const senderMention = `@${username}`;
            const adminNames = admins.map((a) => '@' + a.id.split('@')[0]).join('\n');

            const replyText =
                `${senderMention} telah terdeteksi melakukan penandaan grup ini pada statusnya!\n\n` +
                `${adminNames}`;

            const mentions = [participant, ...adminMentions];

            await sock.sendMessage(id, {
                text: replyText,
                mentions: mentions,
            });
        } catch (err) {
            console.error("Gagal memproses metadata:", err.message);
        }
    }
};

module.exports.SELF = false;
```