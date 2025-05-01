<title>Country</title>
<desc>Mendapatkan data tentang negara</desc>
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
- Kirim pesan teks ke bot dengan format: `.country <negara>`

---

```
const axios = require('axios');

const EMOJIS = {
    loading: '🕒',
    success: '✅',
    error: '❌'
};

module.exports = async (sock, message, msg, sender) => {
    if (msg && msg.toLowerCase().startsWith('.country')) {
        const countryName = msg.split(' ').slice(1).join(' ').trim();

        if (!countryName) {
            await sock.sendMessage(sender, { text: 'Contoh: .country indonesia' }, { quoted: message });
            return;
        }

        try {
			await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });
            const { data } = await axios.get(`https://restcountries.com/v3.1/name/${countryName}`);
            const country = data[0];
            const { flag, name, capital, region, subregion, population, currencies, timezones, cca2, ccn3, cca3, cioc, maps } = country;
            const currencyText = currencies
                ? Object.values(currencies)
                    .map(c => `${c.name} (${c.symbol})`)
                    .join(', ')
                : 'N/A';

            const responseText = `
*${flag} ${name.common} (${name.official})*

📍 *Capital:* ${capital?.[0] || 'N/A'}
🌍 *Region:* ${region}
🌐 *Subregion:* ${subregion}
👥 *Population:* ${population.toLocaleString()}
💰 *Currency:* ${currencyText}
🕒 *Timezones:* ${timezones?.join(', ') || 'N/A'}

🔤 *Codes:*
• CCA2: ${cca2}
• CCN3: ${ccn3}
• CCA3: ${cca3}
• CIOC: ${cioc}

🗺️ *Map:* Google Maps: ${maps.googleMaps}
            `.trim();

            await sock.sendMessage(sender, { text: responseText }, { quoted: message });
			await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });
        } catch (err) {
            console.error('Country Error:', err.message);
		await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
        }
    }
};
```