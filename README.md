# ✨ LuminaAgent

Personal AI Agent berbasis [nanobot-ai](https://github.com/HKUDS/nanobot), di-fork dan di-rename dengan branding Lumina. Berjalan di terminal, WebUI browser, dan Telegram bot.

## Fitur

- 💬 **CLI Chat** — chat langsung di terminal
- 🌐 **WebUI** — tampilan chat di browser (`localhost:8080`)
- 🤖 **Telegram Bot** — agent via Telegram
- 🔧 **19 Tools** — shell exec, file read/write, web search, web fetch, cron, MCP, dan lainnya
- 🧠 **Memory** — ingat percakapan, auto-consolidation
- ⚡ **Multi-provider** — OpenRouter, Anthropic, OpenAI, Gemini, Ollama, dll

---

## Requirement

- Android dengan **Termux** (F-Droid recommended)
- **proot-distro** Ubuntu
- Python 3.11+
- API Key (OpenRouter / Anthropic / lainnya)

---

## Instalasi di Termux

### 1. Install Termux & proot-distro

Download Termux dari [F-Droid](https://f-droid.org/packages/com.termux/), bukan Play Store.

```bash
# Di Termux
pkg update && pkg upgrade -y
pkg install proot-distro -y
proot-distro install ubuntu
proot-distro login ubuntu
```

### 2. Setup Ubuntu

```bash
# Di dalam Ubuntu
apt update && apt install -y python3 python3-pip git curl unzip
```

### 3. Install LuminaAgent

```bash
pip install nanobot-ai --break-system-packages
```

### 4. Rename binary ke `lumina`

```bash
cp /usr/local/bin/nanobot /usr/local/bin/lumina
chmod +x /usr/local/bin/lumina
```

### 5. Inisialisasi config

```bash
lumina onboard
```

### 6. Set API Key

```bash
# Buka config
nano /root/.nanobot/config.json
```

Cari bagian `"openrouter"` dan isi:

```json
"openrouter": {
  "apiKey": "sk-or-v1-...",
  "apiBase": "https://openrouter.ai/api/v1",
  "apiType": "auto",
  "extraHeaders": null,
  "extraBody": null
}
```

Atau pakai script otomatis:

```bash
python3 -c "
import json
with open('/root/.nanobot/config.json') as f: c = json.load(f)
c['providers']['openrouter']['apiKey'] = 'sk-or-v1-KEYMU'
c['providers']['openrouter']['apiBase'] = 'https://openrouter.ai/api/v1'
c['agents']['defaults']['model'] = 'google/gemini-2.0-flash-001'
c['agents']['defaults']['provider'] = 'openrouter'
c['agents']['defaults']['botName'] = 'Lumina'
c['agents']['defaults']['botIcon'] = '✨'
with open('/root/.nanobot/config.json', 'w') as f: json.dump(c, f, indent=2)
print('done')
"
```

### 7. Enable WebUI

```bash
python3 -c "
import json
with open('/root/.nanobot/config.json') as f: c = json.load(f)
c['channels'] = {'websocket': {'port': 8080, 'enabled': True}}
with open('/root/.nanobot/config.json', 'w') as f: json.dump(c, f, indent=2)
print('done')
"
```

### 8. Jalankan

**CLI Chat:**
```bash
lumina agent
```

**WebUI (buka browser ke `http://localhost:8080`):**
```bash
lumina gateway --port 8081
```

---

## Cara Dapat API Key

### OpenRouter (Recommended — gratis ada)
1. Daftar di [openrouter.ai](https://openrouter.ai)
2. Masuk ke [openrouter.ai/keys](https://openrouter.ai/keys)
3. Buat key baru → copy

### Anthropic
1. Daftar di [console.anthropic.com](https://console.anthropic.com)
2. API Keys → Create Key

### Google Gemini (via OpenRouter)
Tidak perlu key Gemini sendiri — cukup OpenRouter, pakai model `google/gemini-2.0-flash-001`.

---

## Setup Telegram Bot

### 1. Buat bot di BotFather
1. Buka Telegram → cari `@BotFather`
2. `/newbot` → ikuti instruksi → copy **token**

### 2. Tambahkan ke config

```bash
python3 -c "
import json
with open('/root/.nanobot/config.json') as f: c = json.load(f)
c['channels']['telegram'] = {
    'enabled': True,
    'token': 'TOKEN_DARI_BOTFATHER'
}
with open('/root/.nanobot/config.json', 'w') as f: json.dump(c, f, indent=2)
print('done')
"
```

### 3. Restart gateway
```bash
pkill -f "lumina gateway"
lumina gateway --port 8081
```

---

## Troubleshooting

### `jiter` build error di Termux (tanpa proot)
```
ERROR: Failed to build 'jiter'
Target triple not supported by rustup
```
**Fix:** Gunakan proot Ubuntu — jauh lebih stabil:
```bash
proot-distro login ubuntu
pip install nanobot-ai --break-system-packages
```

### `externally-managed-environment`
```
error: externally-managed-environment
```
**Fix:** Tambahkan flag:
```bash
pip install nanobot-ai --break-system-packages
```

### `No module named 'tiktoken'`
```bash
pip install tiktoken --break-system-packages
```

### Port already in use
```
OSError: [Errno 98] address already in use
```
**Fix:**
```bash
pkill -f "lumina gateway"
pkill -f "nanobot gateway"
fuser -k 8080/tcp 2>/dev/null
sleep 2
lumina gateway --port 8081
```

WebSocket tetap di port `8080`, health endpoint di port `8081`. Browser ke `http://localhost:8080`.

### API Key 401 Unauthorized
Test key dulu:
```bash
curl https://openrouter.ai/api/v1/chat/completions \
  -H "Authorization: Bearer sk-or-v1-..." \
  -H "Content-Type: application/json" \
  -d '{"model":"google/gemini-2.0-flash-001","messages":[{"role":"user","content":"hi"}]}'
```
Kalau dapat response JSON = key valid. Kalau 401 = key expired/salah → buat key baru.

### WebUI "Not Found"
WebSocket channel belum enabled. Fix:
```bash
python3 -c "
import json
with open('/root/.nanobot/config.json') as f: c = json.load(f)
c['channels']['websocket']['enabled'] = True
with open('/root/.nanobot/config.json', 'w') as f: json.dump(c, f, indent=2)
print('done')
"
```

### Gateway hang / tidak ada output setelah tools registered
Tunggu ~1 menit — websocket channel butuh waktu init. Kalau tetap hang:
```bash
pkill -f "lumina gateway"
lumina gateway --port 8081
```

---

## Struktur Config `/root/.nanobot/config.json`

```json
{
  "agents": {
    "defaults": {
      "model": "google/gemini-2.0-flash-001",
      "provider": "openrouter",
      "botName": "Lumina",
      "botIcon": "✨"
    }
  },
  "providers": {
    "openrouter": {
      "apiKey": "sk-or-v1-...",
      "apiBase": "https://openrouter.ai/api/v1"
    }
  },
  "channels": {
    "websocket": {
      "port": 8080,
      "enabled": true
    },
    "telegram": {
      "token": "TOKEN_BOTFATHER",
      "enabled": true
    }
  }
}
```

---

## Commands

| Command | Deskripsi |
|---|---|
| `lumina agent` | CLI chat interaktif |
| `lumina gateway --port 8081` | Start WebUI + semua channel |
| `lumina onboard` | Setup/init config |
| `lumina status` | Cek status |
| `lumina channels` | Manage channels |
| `lumina provider` | Manage providers |

---

## Credit

Based on [nanobot-ai](https://github.com/HKUDS/nanobot) — MIT License.
