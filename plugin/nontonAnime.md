<title>Nonton Anime</title>
<desc>Mencari semua video episode berdasarkan judul anime</desc>

<!-- Apakah plugin Anda bisa berjalan di windows, linux, atau termux? (supported, not_supported, unknown) -->
<support>
  {
    "windows": "supported",
    "linux": "supported",
    "termux": "supported"
  }
</support>

<!-- Biarkan seperti ini -->
## Instalasi:
Langkah-langkah instalasi pemasangan plugin Anda bisa lihat di [Dokumentasi](/docs#Plugin)
<!-- ################### -->

## Penggunaan:
- Kirim pesan teks ke bot dengan format: `.anime <judul-anime>`

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
    async function searchAnime(query) {
        const searchUrl = `https://v9.animasu.cc/?s=${query}`;
        try {
            const { data } = await axios.get(searchUrl);
            const $ = cheerio.load(data);
            const animeUrl = $('.bs .bsx a').attr('href');
            const animeTitle = $('.bs .bsx .tt').text().trim();
            return { animeTitle, animeUrl };
        } catch (error) {
            console.error('Error fetching search results:', error);
            return null;
        }
    }

    async function getAnimeDetails(animeUrl) {
        try {
            const { data } = await axios.get(animeUrl);
            const $ = cheerio.load(data);
            const episodes = [];

            $('#daftarepisode li').each((index, element) => {
                let episodeUrl = $(element).find('a').attr('href');
                let episodeTitle = $(element).text().trim();
                episodeTitle = episodeTitle.replace('Tonton', '').trim();
                episodes.push({ episodeTitle, episodeUrl });
            });
            return episodes;
        } catch (error) {
            console.error('Error fetching anime details:', error);
            return [];
        }
    }

    async function getEmbedVideoUrl(episodeUrl) {
        try {
            const { data } = await axios.get(episodeUrl);
            const $ = cheerio.load(data);
            const videoUrl = $('video').attr('src') || $('iframe').attr('src');
            return videoUrl || 'URL video tidak ditemukan';
        } catch (error) {
            console.error('Error fetching video URL:', error);
            return 'URL video tidak ditemukan';
        }
    }

    async function scrapeAnime(query) {
        try {
            const { animeTitle, animeUrl } = await searchAnime(query);
            if (!animeTitle || !animeUrl) {
                return { error: 'Anime tidak ditemukan' };
            }
            const episodes = await getAnimeDetails(animeUrl);
            for (let episode of episodes) {
                const videoUrl = await getEmbedVideoUrl(episode.episodeUrl);
                episode.episodeUrl = videoUrl;
            }
            return { animeTitle, episodes };
        } catch (error) {
            console.error('Terjadi kesalahan:', error);
            return { error: 'Terjadi kesalahan saat mengambil data anime' };
        }
    }

    if (msg && msg.toLowerCase().startsWith('.anime')) {
        const query = msg.split(' ').slice(1).join(' ').trim();
        if (!query) {
            await sock.sendMessage(sender, { text: '❌ Masukkan judul anime untuk pencarian!' }, { quoted: message });
            return;
        }
        
        await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });

        const result = await scrapeAnime(query);

        if (result.error) {
            await sock.sendMessage(sender, { text: EMOJIS.error + ' ' + result.error }, { quoted: message });
        } else {
            let response = `Anime: ${result.animeTitle}\n\nEpisodes:\n`;
            result.episodes.forEach((episode, index) => {
                response += `${index + 1}. ${episode.episodeTitle}\nLink: ${episode.episodeUrl}\n\n`;
            });
            
            await sock.sendMessage(sender, { text: response }, { quoted: message });
            await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });
        }
    }
};
```