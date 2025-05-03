<title>OS Info</title>
<desc>Informasi lengkap tentang sistem operasi, termasuk detail CPU, memori, jaringan, GPU, dan penggunaan disk secara real-time.</desc>
<github>fitri-hy</github>
<support>
  {
    "windows": "supported",
    "linux": "supported",
    "termux": "unknown"
  }
</support>

## Instalasi:
Langkah-langkah instalasi pemasangan plugin Anda bisa lihat di [Dokumentasi](/docs#Plugin)

## Penggunaan:
- Kirim pesan teks ke bot dengan format: `.infoos`

---

```js
const os = require('os');
const { execSync } = require('child_process');

const EMOJIS = {
    loading: '🕒',
    success: '✅',
    error: '❌'
};

module.exports = async (sock, message, msg, sender) => {

    if (msg && msg.toLowerCase() === '.infoos') {
        await sock.sendMessage(sender, { react: { text: EMOJIS.loading, key: message.key } });

        const uptimeSec = os.uptime();
        const uptimeHrs = (uptimeSec / 3600).toFixed(2);
        const totalMem = (os.totalmem() / (1024 ** 3)).toFixed(2);
        const freeMem = (os.freemem() / (1024 ** 3)).toFixed(2);
        const usedMem = (totalMem - freeMem).toFixed(2);
        const cpus = os.cpus();
        const osVersion = os.version();
        const nodeVersion = process.version;

        let cpuTemperature = 'N/A';
        if (process.platform === 'linux') {
            try {
                const sensors = execSync('sensors').toString();
                const tempMatch = sensors.match(/Core 0:.*?(\+[\d.]+)°C/);
                if (tempMatch) cpuTemperature = tempMatch[1] + '°C';
            } catch (e) {
                cpuTemperature = 'Gagal mengambil suhu';
            }
        } else if (process.platform === 'win32') {
            cpuTemperature = 'N/A';
        } else if (process.platform === 'darwin') {
            cpuTemperature = 'N/A';
        }

        let diskUsage = 'N/A';
        if (process.platform === 'linux' || process.platform === 'darwin') {
            try {
                diskUsage = execSync('df -h').toString().split('\n')[1].trim();
            } catch (e) {
                diskUsage = 'Gagal mengambil disk usage';
            }
        } else if (process.platform === 'win32') {
            try {
                diskUsage = execSync('wmic logicaldisk get size,freespace,caption').toString().split('\n')[1].trim();
            } catch (e) {
                diskUsage = 'Gagal mengambil disk usage';
            }
        }

        const networkInterfaces = Object.entries(os.networkInterfaces()).map(([name, addrs]) => {
            const ipv4 = addrs.find(a => a.family === 'IPv4');
            return ipv4 ? `  • ${name} \n> ${ipv4.address}` : null;
        }).filter(Boolean).join('\n') || 'Tidak ada antarmuka jaringan yang ditemukan';

        let gpuInfo = 'N/A';
        if (process.platform === 'linux') {
            try {
                gpuInfo = execSync('lspci | grep VGA').toString().trim();
            } catch (e) {
                gpuInfo = 'Gagal mengambil informasi GPU';
            }
        } else if (process.platform === 'darwin') {
            try {
                gpuInfo = execSync('system_profiler SPDisplaysDataType').toString().match(/Chipset Model:.*?(\w.*)/);
                gpuInfo = gpuInfo ? gpuInfo[1] : 'Gagal mengambil informasi GPU';
            } catch (e) {
                gpuInfo = 'Gagal mengambil informasi GPU';
            }
        } else if (process.platform === 'win32') {
            try {
                gpuInfo = execSync('wmic path Win32_VideoController get Caption').toString().split('\n')[1].trim(); // Windows command to get GPU info
            } catch (e) {
                gpuInfo = 'Gagal mengambil informasi GPU';
            }
        }

        const systemDateTime = new Date().toLocaleString();
        const osInfo = `💻 *Informasi Sistem Operasi:*

📌 *Umum*
• Platform
> ${os.platform()}  
• Tipe OS
> ${os.type()}  
• Rilis OS
> ${os.release()}  
• Arsitektur
> ${os.arch()}  
• Hostname
> ${os.hostname()}  
• Direktori Home
> ${os.homedir()}  
• Temp Directory
> ${os.tmpdir()}  
• Waktu Aktif
> ${uptimeHrs} jam (${uptimeSec} detik)  
• Versi OS
> ${osVersion}  
• Versi Node.js
> ${nodeVersion}  
• Tanggal/Waktu
> ${systemDateTime}

🧠 *CPU*
• Model CPU
> ${cpus[0].model}  
• Jumlah Core
> ${cpus.length}  
• Kecepatan CPU
> ${cpus[0].speed} MHz  
• CPU Socket
> ${cpus.length === 1 ? 'Single Socket' : 'Multiple Sockets'}  
• Suhu CPU
> ${cpuTemperature}

🗂️ *Memori*
• RAM Total
> ${totalMem} GB  
• RAM Tersedia
> ${freeMem} GB  
• RAM Digunakan
> ${usedMem} GB  
• Memori Swap
> ${(os.totalmem() - os.freemem() - usedMem).toFixed(2)} GB

🧱 *Jaringan*
• Antarmuka Aktif
${networkInterfaces}

🎮 *VGA/GPU*
• GPU/VGA 
> ${gpuInfo}

📂 *File Sistem*
• Total File System
> ${(totalMem / 1024).toFixed(2)} GB  
• Free Disk Space
> ${(os.freemem() / (1024 ** 3)).toFixed(2)} GB  
• Disk Usage
> ${diskUsage}`.trim();

		await sock.sendMessage(sender, { text: osInfo }, { quoted: message });
		await sock.sendMessage(sender, { react: { text: EMOJIS.success, key: message.key } });
    }
};
```