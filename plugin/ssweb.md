<title>Screenshoot Web</title>
<desc>Ambil tangkapan layar situs web dalam berbagai mode: Desktop, Desktop Halaman Penuh, Seluler, dan Seluler Halaman Penuh.</desc>
<github>fitri-hy</github>
<support>
  {
    "windows": "supported",
    "linux": "supported",
    "termux": "unknown"
  }
</support>

## Instalasi:
Langkah-langkah instalasi pemasangan plugin Anda bisa lihat di [Dokumentasi](/docs#Plugin)

## Penggunaan:
- Kirim pesan teks ke bot dengan format: `.ssweb <url> <mode>`

| Mode | Deskripsi					|
| ---- | ---------------------------|
| `d`  | Desktop					| 
| `df` | Desktop Seluruh Halaman	|
| `m`  | Mobile						|
| `mf` | Mobile Seluruh Halaman		|

---

```
const puppeteer = require('puppeteer');

const EMOJIS = {
    loading: '🕒',
    success: '✅',
    error: '❌'
};

const getDeviceEmulation = (device) => {
    const devices = puppeteer.devices || {}; 
    switch (device) {
        case 'iPhone X':
            return devices['iPhone X'] || {
                viewport: { width: 375, height: 812 },
                userAgent: 'Mozilla/5.0 (iPhone; CPU iPhone OS 13_6_1 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/13.0 Mobile/15E148 Safari/604.1'
            };
        case 'iPhone 12 Pro Max':
            return devices['iPhone 12 Pro Max'] || {
                viewport: { width: 430, height: 932 },
                userAgent: 'Mozilla/5.0 (iPhone; CPU iPhone OS 14_0 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/14.0 Mobile/15E148 Safari/604.1'
            };
        default:
            return {
                viewport: { width: 1280, height: 800 },
                userAgent: 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36'
            };
    }
};

const DEVICE_MODES = {
    fd: {
        viewport: {
            width: 1920,
            height: 1080
        },
        userAgent: 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36'
    },
    d: {
        viewport: {
            width: 1280,
            height: 800
        },
        userAgent: 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36'
    },
    m: getDeviceEmulation('iPhone X'),
    fm: getDeviceEmulation('iPhone 12 Pro Max')
};

module.exports = async (sock, message, msg, sender) => {
    if (msg && msg.toLowerCase().startsWith('.ssweb ')) {
        let [command, url, mode] = msg.split(' ');

        if (!url) {
            return await sock.sendMessage(sender, { text: 'Please provide a valid URL after .ssweb' }, { quoted: message });
        }

        if (!/^https?:\/\//i.test(url)) {
            url = 'http://' + url;
        }

        mode = mode || 'd';

        if (!DEVICE_MODES[mode]) {
            return await sock.sendMessage(sender, { text: 'Invalid mode provided. Available modes: fd, d, m, fm.' }, { quoted: message });
        }

        try {
            await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });

            const browser = await puppeteer.launch();
            const page = await browser.newPage();

            if (mode === 'd' || mode === 'fd') {
                await page.setViewport(DEVICE_MODES[mode].viewport);
                await page.setUserAgent(DEVICE_MODES[mode].userAgent);
            } else {
                await page.emulate(DEVICE_MODES[mode]);
            }

            await page.goto(url);
            
            const fullPage = (mode === 'fd' || mode === 'fm');
            const screenshotBuffer = await page.screenshot({ fullPage });

            await browser.close();

            await sock.sendMessage(sender, {
                image: screenshotBuffer,
                caption: `Screenshot of ${url} in ${mode.toUpperCase()} mode`
            }, { quoted: message });

            await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });

        } catch (error) {
            console.error('Error taking screenshot:', error);
            await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
        }
    }
};
```