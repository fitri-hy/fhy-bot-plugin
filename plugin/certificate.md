<title>Certificate Generator</title>
<desc>Generator sertifikat ini digunakan untuk membuat sertifikat secara otomatis dengan mudah dan cepat.</desc>
<github>fitri-hy</github>
<support>
  {
    "windows": "supported",
    "linux": "supported",
    "termux": "not_supported"
  }
</support>

## Instalasi:
Langkah-langkah instalasi pemasangan plugin Anda bisa lihat di [Dokumentasi](/docs#Plugin)

## Penggunaan:
- Kirim pesan teks ke bot dengan format: `.cert [Namamu] [Nama Perusahaan]`

---

```
const puppeteer = require('puppeteer');
const { delay } = require('@whiskeysockets/baileys');

const EMOJIS = {
    loading: '🕒',
    success: '✅',
    error: '❌'
};

module.exports = async (sock, message, msg, sender) => {
    if (msg.startsWith('.cert')) {
        const match = msg.match(/\.cert\s+\[([^\]]+)\](?:\s+\[([^\]]+)\])?/);

        if (!match) {
            await sock.sendMessage(sender, {
                text: 'Format salah. Contoh penggunaan: `.cert [Namamu] [Nama Perusahaan]`.\nContoh: `.cert [FHY-Bot] [I-As Dev]`',
            }, { quoted: message });
            return;
        }

        const nameRaw = match[1].trim();
        const companyRaw = (match[2] || 'FHY-Bot </>').trim();

        await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });

        const sanitizedName = nameRaw
            .replace(/&/g, "&amp;")
            .replace(/</g, "&lt;")
            .replace(/>/g, "&gt;");

        let sanitizedCompany = companyRaw
            .replace(/&/g, "&amp;")
            .replace(/</g, "&lt;")
            .replace(/>/g, "&gt;");

        if (sanitizedCompany.length > 30) {
            sanitizedCompany = sanitizedCompany.slice(0, 27) + '...';
        }

        const htmlContent = `
        <html>
            <head>
                <script src="https://cdn.tailwindcss.com"></script>
            </head>
            <body class="flex justify-center items-center h-screen w-full">
                <img src="https://raw.githubusercontent.com/fitri-hy/whatsapp-bot-ui/refs/heads/main/public/certi.png" class="-z-50 fixed top-0 w-full h-screen object-cover" />
                <div class="relative w-full flex justify-center items-center">
                    <h1 class="absolute text-center z-50 -mt-20 text-4xl font-bold max-w-lg overflow-hidden line-clamp-1">
                        ${sanitizedName}
                    </h1>
                    <p class="absolute text-center z-50 mt-14 font-italic max-w-sm">
                        In recognition of achievements and dedication in Programming in
                        <span class="font-bold"> ${sanitizedCompany}</span>
                    </p>
                </div>
            </body>
        </html>
        `;

        try {
            const browser = await puppeteer.launch();
            const page = await browser.newPage();
            await page.setContent(htmlContent, { waitUntil: 'networkidle0' });

            const imageBuffer = await page.screenshot({
                type: 'jpeg',
                quality: 90,
                fullPage: false
            });

            await browser.close();

            await delay(1000);

            await sock.sendMessage(sender, {
                image: imageBuffer,
                caption: 'Berikut sertifikatmu. Selamat! 📜🎉',
            }, { quoted: message });

            await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });
        } catch (err) {
            console.error(err);
            await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
        }
    }
};
```