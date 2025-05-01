<title>Requester</title>
<desc>Memungkinkan bot Anda untuk mengirim permintaan HTTP GET dan POST langsung dari chat.</desc>

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
- Kirim pesan teks ke bot dengan format: `.get <endpoint>`, `.post <endpoint> {"key":"value"}`

---

```
const { Mimetype } = require('@whiskeysockets/baileys');
const axios = require('axios');

const EMOJIS = {
  loading: '🕒',
  success: '✅',
  error: '❌',
};

const getUrl = async (url, sender, sock, message) => {
  try {
	await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });
    const response = await axios.get(url);
    await sock.sendMessage(sender, { text: `GET Response: \n${JSON.stringify(response.data, null, 2)}` }, { quoted: message });
	await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });
  } catch (error) {
	await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
  }
};

const postUrl = async (url, data, sender, sock, message) => {
  try {
	await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });
    const response = await axios.post(url, data);
    await sock.sendMessage(sender, { text: `POST Response: \n${JSON.stringify(response.data, null, 2)}` }, { quoted: message });
	await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });
  } catch (error) {
	await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
  }
};

const parseJsonData = (data) => {
  try {
    const cleanedData = data.replace(/\s+/g, ' ').trim();
    if (cleanedData.startsWith('{') && cleanedData.endsWith('}')) {
      return JSON.parse(cleanedData);
    } else {
      throw new Error('Invalid JSON format.');
    }
  } catch (error) {
    throw new Error(`Invalid JSON format: ${error.message}`);
  }
};

module.exports = async (sock, message, msg, sender) => {
  if (msg && msg.toLowerCase().startsWith('.get')) {
    const url = msg.split(' ')[1];
    if (url) {
      await getUrl(url, sender, sock, message);
    } else {
      await sock.sendMessage(sender, { text: 'Please provide a valid URL for the GET request.' }, { quoted: message });
    }
  }

  if (msg && msg.toLowerCase().startsWith('.post')) {
    const [url, ...dataArr] = msg.split(' ').slice(1);
    if (url && dataArr.length > 0) {
      try {
        const data = parseJsonData(dataArr.join(' '));
        await postUrl(url, data, sender, sock, message);
      } catch (error) {
		await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
      }
    } else {
      await sock.sendMessage(sender, { text: 'Please provide both a valid URL and JSON data for the POST request.' }, { quoted: message });
    }
  }
};
```