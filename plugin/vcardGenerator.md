<title>vCard Generator</title>
<desc>Buat kartu kontak digital (vCard) secara instan dengan nama & nomor telepon yang dapat disesuaikan. Cocok untuk membagikan informasi kontak secara profesional maupun pribadi di berbagai perangkat.</desc>
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
- Kirim pesan ke bot dengan format:
```
.vcard
Nama 1|+6281234567890
Nama 2|+6280987654321
```

---

```js
const EMOJIS = {
    loading: '🕒',
    success: '✅',
    error: '❌'
};

module.exports = async (sock, message, msg, sender) => {
    if (msg.startsWith('.vcard')) {
        try {
            await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });
			
            const contactsText = msg.slice(6).trim();

            if (!contactsText) {
                return sock.sendMessage(sender, { text: 'Format salah!\nGunakan:\n.vcard\nNama1|+62xxx\nNama2|+62xxx' }, { quoted: message });
				await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
            }

            const lines = contactsText.split('\n').map(line => line.trim()).filter(Boolean);
            const contactList = [];

            for (const line of lines) {
                const [name, phone] = line.split('|').map(x => x.trim());
                if (!name || !phone) continue;

                const waid = phone.replace('+', '');

                const vcard = `
BEGIN:VCARD
VERSION:3.0
FN:${name}
TEL;type=CELL;type=VOICE;waid=${waid}:${phone}
END:VCARD
`.trim();

                contactList.push({ displayName: name, vcard });
            }

            if (!contactList.length) {
                return sock.sendMessage(sender, { text: 'Tidak ada kontak valid!' }, { quoted: message });
				await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
            }

            await sock.sendMessage(sender, { contacts: { displayName: 'Kontak Dari Bot', contacts: contactList } }, { quoted: message });
            await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });
        } catch (err) {
            await sock.sendMessage(sender, { text: 'Terjadi kesalahan saat membuat vCard.' }, { quoted: message });
            await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
        }
    }
};

module.exports.SELF = false;
```