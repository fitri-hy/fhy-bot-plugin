<title>SEO Alisis</title>
<desc>Menganalisa seo dari sebuah domain</desc>
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
- Kirim pesan teks ke bot dengan format: `.seo <domain>`

---

```
const axios = require('axios');
const cheerio = require('cheerio');

const EMOJIS = {
    loading: '🕒',
    success: '✅',
    error: '❌'
};

module.exports = async (sock, message, msg, sender) => {
    if (msg && msg.toLowerCase().startsWith('.seo')) {
        const domain = msg.split(' ').slice(1).join(' ').trim();

        if (!domain) {
            await sock.sendMessage(sender, { text: 'Contoh: .seo example.com' }, { quoted: message });
            return;
        }

        const url = `https://${domain}`;

        try {
			await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });
            const response = await axios.get(url);
            const $ = cheerio.load(response.data);

            const title = $('title').text().trim() || 'Not Available';
            const metaDescription = $('meta[name="description"]').attr('content') || 'Not Available';
            const metaKeywords = $('meta[name="keywords"]').attr('content') || 'Not Available';
            const ogTitle = $('meta[property="og:title"]').attr('content') || 'Not Available';
            const ogDescription = $('meta[property="og:description"]').attr('content') || 'Not Available';
            const ogImage = $('meta[property="og:image"]').attr('content') || 'Not Available';
            const canonicalUrl = $('link[rel="canonical"]').attr('href') || 'Not Available';

            const metaRobots = $('meta[name="robots"]').attr('content') || 'Not Available';
            const isIndexable = !(metaRobots.includes('noindex'));

            let totalCriteria = 7;
            let totalCriteriaMet = 0;

            if (title !== 'Not Available') totalCriteriaMet++;
            if (metaDescription !== 'Not Available') totalCriteriaMet++;
            if (metaKeywords !== 'Not Available') totalCriteriaMet++;
            if (ogTitle !== 'Not Available') totalCriteriaMet++;
            if (ogDescription !== 'Not Available') totalCriteriaMet++;
            if (ogImage !== 'Not Available') totalCriteriaMet++;
            if (canonicalUrl !== 'Not Available' && isIndexable) totalCriteriaMet++;

            const seoSuccessRate = ((totalCriteriaMet / totalCriteria) * 100).toFixed(2) + ' %';

            const seoReport = `*SEO Report for ${domain}:*\n\n🔍 *Title:*\n> ${title}\n📄 *Meta Description:*\n> ${metaDescription}\n🔑 *Meta Keywords:*\n> ${metaKeywords}\n📸 *OG Title:*\n> ${ogTitle}\n💬 *OG Description:*\n> ${ogDescription}\n🖼️ *OG Image:*\n> ${ogImage}\n🌐 *Canonical URL:*\n> ${canonicalUrl}\n🚦 *Meta Robots:*\n> ${metaRobots}\n✅ *Indexable:*\n> ${isIndexable ? 'Yes' : 'No'}\n\n🔢 *SEO Success Rate:* ${seoSuccessRate}`.trim();

            await sock.sendMessage(sender, { text: seoReport }, { quoted: message });
			await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });
        } catch (err) {
            console.error('SEO Error:', err.message);
			await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
        }
    }
};

```