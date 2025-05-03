<title>Anti Badword & Link</title>
<desc>Mendeteksi dan mencegah penggunaan kata-kata tidak pantas serta tautan yang tidak diizinkan dalam pesan.</desc>
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
- Bot akan otomatis mendeteksi pesan dengan kata-kata buruk & pesan yang mengandung url berdasarkan `blockWords` yang sudah di tentukan. 

---

```
module.exports = async (sock, message, msg, sender) => {
    const messageBody = message.message?.conversation || message.message?.extendedTextMessage?.text || '';

    const blockWords = [
		'bit.ly', 't.me', 'chat.whatsapp.com', 'tai', 'goblok', 'kontol', 'babi', 'anjing', 'memek', 'sialan', 'pecah', 'setan'
	];

    const containsBlockWords = (text) => {
        return blockWords.some(word => new RegExp(`\\b${word}\\b`, 'i').test(text));
    };

    const remoteJid = message.key?.remoteJid;
    const messageKey = message.key;

    if (remoteJid && containsBlockWords(messageBody)) {
        try {
            await sock.sendMessage(remoteJid, { delete: messageKey });
        } catch (error) {
            console.error('Gagal menghapus pesan:', error);
        }
    }
};

module.exports.SELF = false;
```