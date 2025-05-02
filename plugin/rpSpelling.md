<title>RP Spelling</title>
<desc>Penyebutan mata uang rupiah dalam bahasa indonesia atau ingris.</desc>
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
- Kirim pesan teks ke bot dengan format: `.translate <kode_bahasa> <teks>`

---

```
const EMOJIS = {
    loading: '🕒',
    success: '✅',
    error: '❌'
};

function formatCurrency(amount) {
    return new Intl.NumberFormat('id-ID', {
        minimumFractionDigits: 2,
        maximumFractionDigits: 2
    }).format(amount);
}

function numberToWords(number, lang = 'id') {
    const units = lang === 'en' 
        ? ["", "One", "Two", "Three", "Four", "Five", "Six", "Seven", "Eight", "Nine"] 
        : ["", "Satu", "Dua", "Tiga", "Empat", "Lima", "Enam", "Tujuh", "Delapan", "Sembilan"];
    const tens = lang === 'en' 
        ? ["", "Ten", "Twenty", "Thirty", "Forty", "Fifty", "Sixty", "Seventy", "Eighty", "Ninety"] 
        : ["", "Sepuluh", "Dua Puluh", "Tiga Puluh", "Empat Puluh", "Lima Puluh", "Enam Puluh", "Tujuh Puluh", "Delapan Puluh", "Sembilan Puluh"];
    const teens = lang === 'en' 
        ? ["Ten", "Eleven", "Twelve", "Thirteen", "Fourteen", "Fifteen", "Sixteen", "Seventeen", "Eighteen", "Nineteen"] 
        : ["Sepuluh", "Sebelas", "Dua Belas", "Tiga Belas", "Empat Belas", "Lima Belas", "Enam Belas", "Tujuh Belas", "Delapan Belas", "Sembilan Belas"];
    const scales = lang === 'en' 
        ? ["", "Thousand", "Million", "Billion", "Trillion", "Quadrillion"] 
        : ["", "Ribu", "Juta", "Miliar", "Triliun", "Kuadriliun"];

    if (number === 0) return lang === 'en' ? "Zero" : "Nol";

    let words = "";
    let scaleIndex = 0;

    while (number > 0) {
        let chunk = number % 1000;

        if (chunk > 0) {
            let chunkWords = "";

            if (chunk > 99) {
                const hundreds = Math.floor(chunk / 100);
                chunkWords += units[hundreds] + (lang === 'en' ? " Hundred " : " Ratus ");
                chunk %= 100;
            }

            if (chunk > 19) {
                const tensPlace = Math.floor(chunk / 10);
                chunkWords += tens[tensPlace] + " ";
                chunk %= 10;
            } else if (chunk > 9) {
                chunkWords += teens[chunk - 10] + " ";
                chunk = 0;
            }

            if (chunk > 0) {
                chunkWords += units[chunk] + " ";
            }

            chunkWords += scales[scaleIndex] + " ";
            words = chunkWords + words;
        }

        number = Math.floor(number / 1000);
        scaleIndex++;
    }

    let finalWords = words.trim();

    if (lang === 'id') {
        finalWords = finalWords
            .replace(/\bSatu Ratus\b/gi, 'Seratus')
            .replace(/\bSatu Ribu\b/gi, 'Seribu');
    }

    return finalWords;
}


function parseAmount(amount) {
    const normalized = amount.replace(/\./g, '').replace(',', '.');
    return parseFloat(normalized);
}

module.exports = async (sock, message, msg, sender) => {

    if (msg && msg.toLowerCase().startsWith('.spell')) {
        try {
            await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });

            const parts = msg.split(' ');
            const lang = parts[1] || 'id';
            const rawAmount = parts.slice(2).join(' ').trim();

            if (!rawAmount || isNaN(parseAmount(rawAmount))) {
                await sock.sendMessage(sender, { text: `Format salah. Gunakan: .spell <id/en> <nominal>` }, { quoted: message });
                return;
            }

            const numericValue = parseAmount(rawAmount);
            const formattedAmount = formatCurrency(numericValue);
            const spelledOut = numberToWords(Math.floor(numericValue), lang) + 
                (Math.round((numericValue % 1) * 100) > 0 
                    ? ` ${numberToWords(Math.round((numericValue % 1) * 100), lang)} ${lang === 'en' ? "Cents" : "Sen"}` 
                    : "") + 
                (lang === 'en' ? " Rupiah" : " Rupiah");

            await sock.sendMessage(sender, { text: `💰 *${formattedAmount}* \n> ${spelledOut}` }, { quoted: message });
            await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });

        } catch (err) {
            console.error(err);
            await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
        }
    }
};
```