<title>Github Stalker</title>
<desc>Lihat profil, repo terbaru, bintang, dan fork dari akun GitHub hanya dengan satu perintah.</desc>
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
- Kirim pesan teks ke bot dengan format: `.github <username>`

---

```
const { Octokit } = require('@octokit/rest');
const octokit = new Octokit();
const axios = require('axios');

const EMOJIS = {
    loading: '🕒',
    success: '✅',
    error: '❌'
};

module.exports = async (sock, message, msg, sender) => {
	if (msg && msg.toLowerCase().startsWith('.github')) {
		await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });

		const parts = msg.split(' ');
		if (parts.length < 2) {
			await sock.sendMessage(sender, { text: 'Masukkan username GitHub.\nContoh: .github torvalds' }, { quoted: message });
			return;
		}

		const username = parts[1];

		try {
			const { data } = await octokit.rest.users.getByUsername({ username });

			const reposResponse = await octokit.rest.repos.listForUser({
				username,
				sort: 'updated',
				per_page: 5
			});

			const repos = reposResponse.data;

			let profileText = `👤 *GitHub Profile*\n\n`;
			profileText += `*Name:* ${data.name || 'N/A'}\n`;
			profileText += `*Username:* ${data.login}\n`;
			profileText += `*Bio:* ${data.bio || 'N/A'}\n`;
			profileText += `*Company:* ${data.company || 'N/A'}\n`;
			profileText += `*Location:* ${data.location || 'N/A'}\n`;
			profileText += `*Public Repos:* ${data.public_repos}\n`;
			profileText += `*Followers:* ${data.followers}\n`;
			profileText += `*Following:* ${data.following}\n`;
			profileText += `*Profile Link:* ${data.html_url}\n\n`;

			if (repos.length > 0) {
				profileText += `📦 *5 Repo Terbaru:*\n`;
				repos.forEach((repo, i) => {
					profileText += `\n${i + 1}. *${repo.name}*\n`;
					if (repo.description) profileText += `📝 ${repo.description}\n`;
					profileText += `⭐ Stars: ${repo.stargazers_count}   🍴 Forks: ${repo.forks_count}\n`;
					profileText += `🔗 ${repo.html_url}\n`;
				});
			} else {
				profileText += `📦 User ini belum punya repo publik.`;
			}


			const profilePic = data.avatar_url;

			const imgRes = await axios.get(profilePic, { responseType: 'arraybuffer' });
			const imgBuffer = Buffer.from(imgRes.data, 'binary');

			await sock.sendMessage(sender, {
				image: imgBuffer,
				caption: profileText
			}, { quoted: message });

			await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });

		} catch (error) {
			console.error(error);
			await sock.sendMessage(sender, { text: '❌ Gagal mengambil data. Pastikan username valid.' }, { quoted: message });
			await sock.sendMessage(sender, { react: { text: EMOJIS.error, key: message.key } });
		}

	}
};
```