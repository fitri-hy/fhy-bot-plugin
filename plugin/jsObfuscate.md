<title>Github Stalker</title>
<desc>Lihat profil, repo terbaru, bintang, dan fork dari akun GitHub hanya dengan satu perintah.</desc>
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
- Kirim pesan teks ke bot dengan format: `.obfuscate <kode>`
- Kirim kutipan ke pesan yang berisi kode JavaScript: `.obfuscate`

---

```
const JavaScriptObfuscator = require('javascript-obfuscator');

const EMOJIS = {
    loading: '🕒',
    success: '✅',
    error: '❌'
};

module.exports = async (sock, message, msg, sender) => {
    if (msg && msg.toLowerCase().startsWith('.obfuscate')) {
        await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });

        const quotedMsg = message.message?.extendedTextMessage?.contextInfo?.quotedMessage;
        const codeText = quotedMsg?.conversation || quotedMsg?.extendedTextMessage?.text || msg.replace('.obfuscate', '').trim();

        if (!codeText) {
            await sock.sendMessage(sender, {
                text: 'Kirim `.obfuscate` <kode> atau reply ke pesan yang berisi kode JavaScript.',
            }, { quoted: message });
            return;
        }

        const isLikelyJS = /function|const|let|var|=>|[{;]/.test(codeText);

        if (!isLikelyJS) {
            await sock.sendMessage(sender, {
                text: 'Kode tidak dikenali sebagai JavaScript.',
            }, { quoted: message });
            return;
        }

        try {
            const obfuscated = JavaScriptObfuscator.obfuscate(codeText, {
                compact: true,
                controlFlowFlattening: true,
                controlFlowFlatteningThreshold: 0.75,
                numbersToExpressions: true,
                simplify: true,
                stringArray: true,
                stringArrayEncoding: ['base64'],
                stringArrayThreshold: 0.75,
            });

            const result = obfuscated.getObfuscatedCode();

            await sock.sendMessage(sender, {
                text: `🔐 *Kode setelah di-obfuscate:*\n\n\`\`\`js\n${result}\n\`\`\``
            }, { quoted: message });

            await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });

        } catch (err) {
            console.error(err);
            await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
        }
    }
};
```