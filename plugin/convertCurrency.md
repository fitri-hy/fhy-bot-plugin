<title>Konversi Mata Uang</title>
<desc>Konversikan mata uang dari satu negara ke negara lainnya dengan mudah.</desc>
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
- Kirim pesan teks ke bot dengan format: `.convert <jumlah> <mata uang asal> <mata uang tujuan>`

---

```
const { Mimetype } = require('@whiskeysockets/baileys');

const EMOJIS = {
    loading: '🕒',
    success: '✅',
    error: '❌'
};

const EXCHANGE_RATES = {
    USD: 1,
    EUR: 0.93,
    IDR: 16430,
    GBP: 0.76,
    JPY: 133
};

module.exports = async (sock, message, msg, sender) => {
    if (msg && msg.toLowerCase().startsWith('.convert ')) {
        try {
            await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });

            const input = msg.slice(9).trim();
            const [amount, fromCurrency, toCurrency] = input.split(' ');

            if (!amount || !fromCurrency || !toCurrency) {
                throw new Error('Format yang benar: .convert <jumlah> <mata uang asal> <mata uang tujuan>');
            }

            if (isNaN(amount)) {
                throw new Error('Jumlah yang dimasukkan harus berupa angka.');
            }

            const amountValue = parseFloat(amount);

            if (!EXCHANGE_RATES[fromCurrency.toUpperCase()] || !EXCHANGE_RATES[toCurrency.toUpperCase()]) {
                throw new Error('Mata uang tidak valid. Pastikan Anda menggunakan kode mata uang yang benar.');
            }

            const rateFrom = EXCHANGE_RATES[fromCurrency.toUpperCase()];
            const rateTo = EXCHANGE_RATES[toCurrency.toUpperCase()];
            const convertedAmount = (amountValue * rateTo) / rateFrom;
            const resultMessage = `${EMOJIS.success} Konversi Mata Uang Anda:\n\n${amountValue} ${fromCurrency.toUpperCase()} = ${convertedAmount.toFixed(2)} ${toCurrency.toUpperCase()}`;

            await sock.sendMessage(sender, { text: resultMessage }, { quoted: message });
            await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });

        } catch (error) {
            await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
            const errorMessage = error.message || 'Terjadi kesalahan, harap coba lagi.';
            await sock.sendMessage(sender, { text: errorMessage, quoted: message });
        }
    }
};
```