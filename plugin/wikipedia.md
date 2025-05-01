<title>Wikipedia</title>
<desc>Pencarian konten dari wikipedia</desc>
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
- Kirim pesan teks ke bot dengan format: `.wiki-ai <text>`, `.wiki-search <text>`, `.wiki-image <text>`

---

```
const axios = require('axios');

const EMOJIS = {
    loading: '🕒',
    success: '✅',
    error: '❌'
};

module.exports = async (sock, message, msg, sender) => {
    if (msg && msg.toLowerCase().startsWith('.wiki-ai')) {
        const query = msg.split(' ').slice(1).join(' ').trim();

        if (!query) {
            await sock.sendMessage(sender, { text: 'Contoh: .wiki-ai Jakarta' }, { quoted: message });
            return;
        }

        try {
			await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });
            const wikiApiUrl = `https://id.wikipedia.org/w/api.php?action=query&list=search&srsearch=${encodeURIComponent(query)}&format=json&utf8=1&origin=*`;
            const response = await axios.get(wikiApiUrl);
            const searchResults = response.data.query.search;

            if (searchResults.length > 0) {
                const result = searchResults[0];
                const responseMessage = `*${result.title}*\n\n${result.snippet.replace(/<[^>]+>/g, '')}...\n\nBaca lebih lanjut di: https://id.wikipedia.org/wiki/${encodeURIComponent(result.title)}`;

                await sock.sendMessage(sender, { text: responseMessage }, { quoted: message });
            } else {
                await sock.sendMessage(sender, { text: 'Data tidak ditemukan.' }, { quoted: message });
            }
			await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });
        } catch (error) {
            console.error('Error:', error);
			await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
        }
    }

    if (msg && msg.toLowerCase().startsWith('.wiki-search')) {
        const query = msg.split(' ').slice(1).join(' ').trim();

        if (!query) {
            await sock.sendMessage(sender, { text: 'Contoh: .wiki-search Jakarta' }, { quoted: message });
            return;
        }
		
        try {
			await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });
            const wikiApiUrl = `https://id.wikipedia.org/w/api.php?action=query&list=search&srsearch=${encodeURIComponent(query)}&srlimit=5&format=json&utf8=1&origin=*`;
            const response = await axios.get(wikiApiUrl);
            const searchResults = response.data.query.search;

            if (searchResults.length > 0) {
                let responseMessage = "Hasil pencarian:\n\n";

                for (const result of searchResults) {
                    const title = result.title;
                    const snippet = result.snippet.replace(/<[^>]+>/g, '');
                    responseMessage += `*${title}*\n${snippet}...\nBaca lebih lanjut di: https://id.wikipedia.org/wiki/${encodeURIComponent(title)}\n\n`;
                }

                await sock.sendMessage(sender, { text: responseMessage }, { quoted: message });
            } else {
                await sock.sendMessage(sender, { text: 'Data tidak ditemukan.' }, { quoted: message });
            }
			await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });
        } catch (error) {
            console.error('Error:', error);
			await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
        }
    }

	if (msg && msg.toLowerCase().startsWith('.wiki-image')) {
		const query = msg.split(' ').slice(1).join(' ').trim();

		if (!query) {
			await sock.sendMessage(sender, { text: 'Contoh: .wiki-image Jakarta' }, { quoted: message });
			return;
		}

		try {
			await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });
			const searchResponse = await axios.get(`https://id.wikipedia.org/w/api.php`, {
				params: {
					action: 'query',
					format: 'json',
					list: 'search',
					srsearch: query,
					utf8: 1,
					srlimit: 1
				}
			});
			const pageId = searchResponse.data.query.search[0]?.pageid;
			const title = searchResponse.data.query.search[0]?.title;
			const snippet = searchResponse.data.query.search[0]?.snippet.replace(/<[^>]+>/g, '');
			if (!pageId) {
				await sock.sendMessage(sender, { text: 'Tidak ditemukan gambar untuk pencarian ini.' }, { quoted: message });
				return;
			}

			const imageResponse = await axios.get(`https://id.wikipedia.org/w/api.php`, {
				params: {
					action: 'query',
					format: 'json',
					prop: 'pageimages|imageinfo',
					pageids: pageId,
					pilimit: 'max',
					pithumbsize: 500
				}
			});

			const imageUrl = imageResponse.data.query.pages[pageId]?.thumbnail?.source;
			const originalImageUrl = imageResponse.data.query.pages[pageId]?.imageinfo?.[0]?.url;
			const finalImageUrl = originalImageUrl || imageUrl;

			if (!finalImageUrl) {
				await sock.sendMessage(sender, { text: 'Tidak ada gambar ditemukan.' }, { quoted: message });
				return;
			}

			const responseMessage = `*${title}*\n\n${snippet}...\n\nBaca lebih lanjut di: https://id.wikipedia.org/wiki/${encodeURIComponent(title)}`;

			await sock.sendMessage(sender, { image: { url: finalImageUrl }, caption: responseMessage }, { quoted: message });
			await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });
		} catch (error) {
			console.error("Error fetching Wikipedia image:", error);
			await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
		}
	}
};
```