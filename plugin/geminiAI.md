<title>Gemini AI</title>
<desc>Gemini AI untuk memberikan respons cerdas dalam interaksi</desc>
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
- Kirim pesan dengan kutip gambar ke bot dengan format: `.gemini-analyze <pertanyaan>`
- Kirim pesan teks ke bot dengan format: `.gemini-chat <pertanyaan>`

---

```js
const { GoogleGenerativeAI } = require('@google/generative-ai');
const { downloadContentFromMessage, Mimetype } = require('@whiskeysockets/baileys');

const EMOJIS = {
    loading: '🕒',
    success: '✅',
    error: '❌'
};

const genAI = new GoogleGenerativeAI('Enter_GeminiAI_APIKEY_Here');

module.exports = async (sock, message, msg, sender) => {

    if (msg && msg.toLowerCase().startsWith('.gemini-chat')) {
        const userMessage = msg.slice(7).trim();
        
        if (!userMessage) {
            await sock.sendMessage(sender, { text: 'Please provide a message after `.chat <your message>`.' }, { quoted: message });
            return;
        }

        await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });

        try {
            const model = genAI.getGenerativeModel({ model: "gemini-1.5-flash" });
            const result = await model.generateContent(userMessage);
            const response = await result.response;
            const generatedText = response.text();
            await sock.sendMessage(sender, { text: generatedText }, { quoted: message });
            await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });
        } catch (error) {
            console.error('Error calling Gemini AI:', error);
            await sock.sendMessage(sender, { text: 'There was an error while contacting Gemini AI. Please try again later.' }, { quoted: message });
            await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
        }
    }
	
	if (msg && msg.toLowerCase().startsWith('.gemini-analyze')) {
        const quotedMessage = message.message?.extendedTextMessage?.contextInfo?.quotedMessage;
        const caption = msg.slice(17).trim();

        if (!quotedMessage) {
            await sock.sendMessage(sender, { text: 'Please quote an image with `.gemini-analyze <your question>`.' }, { quoted: message });
            return;
        }

        if (!caption) {
            await sock.sendMessage(sender, { text: 'Please include a question after `.gemini-analyze`.' }, { quoted: message });
            return;
        }

        const imageMessage = quotedMessage?.imageMessage || quotedMessage?.videoMessage;
        if (!imageMessage || !imageMessage.mimetype.startsWith('image/')) {
            await sock.sendMessage(sender, { text: 'The quoted message does not contain a valid image.' }, { quoted: message });
            return;
        }

        await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });

        try {
            const stream = await downloadContentFromMessage(imageMessage, 'image');
            let buffer = Buffer.alloc(0);
            for await (const chunk of stream) {
                buffer = Buffer.concat([buffer, chunk]);
            }

            const base64Image = buffer.toString('base64');

            const imagePart = {
                inlineData: {
                    data: base64Image,
                    mimeType: imageMessage.mimetype,
                },
            };

            const model = genAI.getGenerativeModel({ model: 'gemini-1.5-flash' });
            const aiResult = await model.generateContent([caption, imagePart]);
            const response = await aiResult.response;
            const generatedText = response.text();

            await sock.sendMessage(sender, { text: generatedText }, { quoted: message });
            await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });

        } catch (error) {
            console.error('Error while analyzing the image:', error);
            await sock.sendMessage(sender, {
                text: `Error: ${error.message || 'Something went wrong while analyzing the image.'}`
            }, { quoted: message });
            await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
        }
    }
};
```