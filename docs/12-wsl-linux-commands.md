# 12 — Command WSL/Linux untuk Pemula

Tujuan: lo belum pernah pake Linux. Ini semua command dasar yang lo butuh buat jalanin Hermes Agent di WSL.

---

## 1. Apa itu WSL

**WSL** = Windows Subsystem for Linux. Intinya: lo jalanin Linux di dalam Windows, tanpa dual-boot, tanpa VM berat. Kayak punya "terminal Linux" di dalam laptop Windows lo.

Hermes Agent butuh Linux. WSL2 itu cara paling gampang kalau lo pake Windows.

---

## 2. Install WSL2 (sekali aja)

Buka **PowerShell sebagai Admin**, jalanin:

```powershell
wsl --install
```

Restart laptop. Setelah restart, buka "Ubuntu" dari Start Menu. Set username + password. Done.

---

## 3. Command Navigasi (kayak File Explorer, tapi pake ketik)

| Command | Fungsi | Analogi |
|---------|--------|---------|
| `pwd` | Tampilkan lokasi lo sekarang | "Gw lagi di folder mana?" |
| `ls` | Liat isi folder | Buka folder di Explorer |
| `ls -la` | Liat isi folder + detail (ukuran, permission, hidden files) | Properties di Explorer |
| `cd nama-folder` | Masuk ke folder | Double-click folder |
| `cd ..` | Naik 1 level ke atas | Klik "Back" |
| `cd ~` | Balik ke home directory | Balik ke "C:\Users\lo" |
| `cd /` | Ke root (paling atas) | Ke "This PC" |

### Contoh:

```bash
pwd                    # output: /home/hataf
ls                     # output: Documents  Downloads  .hermes
cd .hermes             # masuk ke folder .hermes
ls                     # liat isi: config.yaml  .env  SOUL.md  skills/
cd ..                  # balik ke /home/hataf
```

---

## 4. Command File (bikin, hapus, copy, pindah)

| Command | Fungsi | Analogi |
|---------|--------|---------|
| `cat file.txt` | Tampilkan isi file | Buka Notepad, baca |
| `nano file.txt` | Edit file (editor simpel) | Notepad |
| `touch file.txt` | Bikin file kosong baru | Klik kanan → New File |
| `mkdir nama-folder` | Bikin folder baru | Klik kanan → New Folder |
| `cp file.txt backup.txt` | Copy file | Ctrl+C → Ctrl+V |
| `cp -r folder/ backup/` | Copy folder + isinya | Copy folder |
| `mv file.txt /path/lain/` | Pindah file | Cut → Paste |
| `mv nama-lama nama-baru` | Rename file/folder | Rename |
| `rm file.txt` | Hapus file (GAK BISA UNDO!) | Delete permanent |
| `rm -r folder/` | Hapus folder + isinya (HATI-HATI!) | Delete folder permanent |

### ⚠️ PERINGATAN:

```bash
# JANGAN PERNAH jalanin ini:
rm -rf /              # HAPUS SEMUA. Seluruh sistem. Gak bisa balik.
rm -rf ~              # Hapus semua file di home lo.
```

Kalau ragu, tambahin `-i` (interactive = tanya dulu sebelum hapus):

```bash
rm -ri folder/        # dia tanya "yakin?" per file
```

---

## 5. Command Hermes yang lo perlu

| Command | Fungsi |
|---------|--------|
| `hermes` | Buka TUI (chat di terminal) |
| `hermes --version` | Cek versi |
| `hermes doctor` | Diagnose masalah |
| `hermes update` | Update ke versi terbaru |
| `hermes model` | Pilih/ganti model |
| `hermes config edit` | Edit config di editor |
| `hermes gateway` | Jalanin Telegram bot (foreground) |
| `hermes gateway install` | Install sebagai service (background) |
| `hermes cron list` | Liat semua cronjob |
| `hermes skills list` | Liat semua skill terinstall |
| `hermes profile create nama` | Bikin profile baru |

---

## 6. Command Sistem (cek kondisi server/VPS)

| Command | Fungsi | Kapan pake |
|---------|--------|-----------|
| `df -h` | Cek disk usage | Mau tau sisa storage |
| `free -h` | Cek RAM usage | Kalau Hermes lambat |
| `top` atau `htop` | Monitor process real-time | Cek apa yang makan resource |
| `uptime` | Berapa lama nyala | Habis reboot? |
| `whoami` | Username lo siapa | Konfirmasi user |
| `uname -a` | Info OS + kernel | Debugging |

---

## 7. Command Service (buat gateway yang jalan background)

| Command | Fungsi |
|---------|--------|
| `sudo systemctl start hermes-agent` | Start service |
| `sudo systemctl stop hermes-agent` | Stop service |
| `sudo systemctl restart hermes-agent` | Restart |
| `sudo systemctl status hermes-agent` | Cek status (running? error?) |
| `sudo systemctl enable hermes-agent` | Auto-start saat boot |
| `journalctl -u hermes-agent -f` | Liat log real-time (Ctrl+C buat stop) |

> `sudo` = "run as admin". Pertama kali pake bakal minta password lo.

---

## 8. Command Network (cek koneksi)

| Command | Fungsi | Kapan pake |
|---------|--------|-----------|
| `ping google.com` | Cek internet nyambung | Kalau Hermes gak bisa connect |
| `curl -I https://api.deepseek.com` | Test API endpoint | Verify endpoint bisa diakses |
| `ip addr` | Liat IP address lo | Kalau setting network |
| `ss -tlnp` | Liat port yang aktif | Cek service jalan di port berapa |

---

## 9. Command Package (install software)

| Command | Fungsi |
|---------|--------|
| `sudo apt update` | Refresh daftar package (JALANIN DULU sebelum install apapun) |
| `sudo apt install nama-package` | Install software |
| `sudo apt upgrade` | Update semua software |
| `pip install nama-package` | Install Python package |
| `pip install --upgrade nama` | Update Python package |
| `which nama` | Cek apakah software terinstall (output path = ada) |

### Yang lo perlu install buat Hermes:

```bash
sudo apt update
sudo apt install -y ffmpeg curl git
pip install faster-whisper yt-dlp
```

---

## 10. Command Git (buat backup/version control)

| Command | Fungsi |
|---------|--------|
| `git status` | Cek ada perubahan apa |
| `git add .` | Stage semua perubahan |
| `git commit -m "pesan"` | Simpan snapshot |
| `git push` | Upload ke GitHub |
| `git pull` | Download update dari GitHub |
| `git log --oneline -5` | Liat 5 commit terakhir |

---

## 11. Shortcut keyboard di terminal

| Shortcut | Fungsi |
|----------|--------|
| `Ctrl+C` | Cancel/stop command yang lagi jalan |
| `Ctrl+D` | Keluar dari terminal |
| `Ctrl+L` | Clear layar (sama kayak ketik `clear`) |
| `Tab` | Auto-complete nama file/folder |
| `↑` / `↓` | Browse command history |
| `Ctrl+R` | Search command history |
| `Ctrl+A` | Cursor ke awal baris |
| `Ctrl+E` | Cursor ke akhir baris |

---

## 12. File permission (kenapa kadang "Permission denied")

Di Linux, setiap file punya permission: siapa yang boleh baca/tulis/jalanin.

```bash
ls -la ~/.hermes/.env
# output: -rw------- 1 hataf hataf 512 May 22 .env
#          ^^^
#          rw- = owner bisa read+write
#          --- = group gak bisa apa-apa
#          --- = others gak bisa apa-apa
```

Kalau dapet "Permission denied":

```bash
chmod 600 ~/.hermes/.env         # cuma owner yang bisa baca/tulis (untuk secrets)
chmod 755 script.sh              # owner full, others bisa baca+jalanin
chmod +x script.sh               # bikin file bisa di-execute (run as program)
```

---

## 13. Environment variable (kenapa penting buat Hermes)

File `.env` lo = kumpulan "variable" yang dibaca Hermes.

```bash
# Liat semua env vars aktif
env

# Liat 1 var spesifik
echo $OPENCODE_GO_API_KEY

# Set var sementara (hilang setelah tutup terminal)
export MY_VAR="hello"

# Set var permanent → tulis di ~/.bashrc atau ~/.hermes/.env
```

---

## 14. Editor di terminal

### Nano (paling gampang buat pemula)

```bash
nano ~/.hermes/config.yaml
```

| Shortcut | Fungsi |
|----------|--------|
| Ketik aja | Edit text |
| `Ctrl+O` lalu Enter | Save |
| `Ctrl+X` | Keluar |
| `Ctrl+W` | Search |
| `Ctrl+K` | Cut baris |
| `Ctrl+U` | Paste baris |

### Vim (buat nanti kalau udah advanced)

Gak usah belajar sekarang. Kalau gak sengaja masuk vim:
- Ketik `:q!` lalu Enter → keluar tanpa save
- Ketik `:wq` lalu Enter → save dan keluar

---

## 15. Path Windows ↔ WSL

Di WSL, drive Windows lo ada di `/mnt/`:

| Windows path | WSL path |
|---|---|
| `C:\Users\lo\Documents` | `/mnt/c/Users/lo/Documents` |
| `D:\projects` | `/mnt/d/projects` |
| Home WSL lo | `~` atau `/home/nama-lo` |

```bash
# Akses file Windows dari WSL:
ls /mnt/c/Users/lo/Downloads/

# Copy file dari Windows ke WSL:
cp /mnt/c/Users/lo/Downloads/file.pdf ~/
```

---

## 16. Troubleshooting umum

| Problem | Fix |
|---------|-----|
| "command not found" | Software belum install. `sudo apt install [nama]` atau cek PATH. |
| "Permission denied" | Tambahin `sudo` di depan, atau `chmod` file-nya. |
| "No such file or directory" | Path salah. Cek dengan `ls` dan `pwd`. |
| Terminal stuck / gak respon | `Ctrl+C` untuk cancel. Kalau masih stuck: tutup tab, buka baru. |
| "apt: command not found" | Lo mungkin bukan di Ubuntu. Cek distro: `cat /etc/os-release`. |
| WSL lambat | Buka Task Manager Windows → cek Vmmem memory usage. Restart WSL: `wsl --shutdown` di PowerShell. |

---

## 17. Workflow harian lo (Hermes di WSL)

```bash
# Pagi — cek semuanya jalan
sudo systemctl status hermes-agent     # gateway Telegram running?
hermes cron status                      # cronjob aktif?

# Edit something
nano ~/.hermes/SOUL.md                  # edit identity
nano ~/.hermes/config.yaml              # edit config

# Restart setelah edit
sudo systemctl restart hermes-agent

# Liat log kalau ada masalah
journalctl -u hermes-agent -f           # Ctrl+C buat stop

# Update Hermes
hermes update

# Backup
tar czf ~/hermes-backup-$(date +%F).tar.gz ~/.hermes/
```

---

## 🔗 Sumber

- Pengetahuan umum Linux command — bukan dari sumber spesifik
- [WSL official docs](https://learn.microsoft.com/en-us/windows/wsl/) — install & troubleshoot WSL
- [Hermes Agent docs](https://hermes-agent.nousresearch.com/docs/) — Hermes CLI commands
