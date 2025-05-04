<title>BMI Kalkulator</title>
<desc>Hitung Indeks Massa Tubuh (BMI) Anda berdasarkan berat badan dan tinggi badan.</desc>
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
- Kirim pesan teks ke bot dengan format: `.bmi <berat badan> <tinggi badan>`

---

```
const EMOJIS = {
    loading: '🕒',
    success: '✅',
    error: '❌'
};

module.exports = async (sock, message, msg, sender) => {
    if (msg && msg.toLowerCase().startsWith('.bmi ')) {
        try {
			await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });

            const input = msg.slice(5).trim();
            const [weight, height] = input.split(' ');

            if (!weight || !height) {
                throw new Error('Berat badan dan tinggi badan harus diisi. Format: .bmi <berat badan> <tinggi badan>');
            }

            if (isNaN(weight) || isNaN(height)) {
                throw new Error('Berat badan dan tinggi badan harus berupa angka.');
            }

            const weightInKg = parseFloat(weight);
            const heightInM = parseFloat(height) / 100;
            if (weightInKg <= 0 || heightInM <= 0) {
                throw new Error('Berat badan dan tinggi badan harus bernilai positif.');
            }

            const bmi = weightInKg / (heightInM * heightInM);

            let category = '';
            if (bmi < 18.5) {
                category = 'Kurus';
            } else if (bmi >= 18.5 && bmi < 24.9) {
                category = 'Normal';
            } else if (bmi >= 25 && bmi < 29.9) {
                category = 'Gemuk';
            } else {
                category = 'Obesitas';
            }

            const resultMessage = `Hasil BMI Anda:\n\nBMI: ${bmi.toFixed(2)}\nKategori: ${category}`;

            await sock.sendMessage(sender, { text: resultMessage }, { quoted: message });
			await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });

        } catch (error) {
			await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
			const errorMessage = error.message || 'Terjadi kesalahan, harap coba lagi.';
            await sock.sendMessage(sender, { text: errorMessage }, { quoted: message });
        }
    }
};
```