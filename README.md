# navi-spotube-plugin

A [Spotube](https://github.com/KRTirtho/spotube) plugin that connects your self-hosted [Navidrome](https://www.navidrome.org/) music server as **both** the track search provider and the audio source.  
All searches and streams come exclusively from your Navidrome instance — no external services are used.

---

## Features

- 🔍 **Track search** via Subsonic `search3.view` — searches only your Navidrome library
- 🎵 **Audio streaming** via Subsonic `stream.view` — direct URL playback from Navidrome
- 🔐 **Subsonic password auth** using the `enc:` hex-encoding scheme (no plain-text password in URLs)
- ✅ **Test connection** — pings your server when you save settings; shows a clear error if the connection fails
- 📀 **Two quality streams** per track:
  - Original file (lossless, no transcoding)
  - MP3 320 kbps (requires ffmpeg configured in Navidrome)
- 💾 **Persistent settings** — credentials are stored in Spotube's local storage across sessions

---

## Requirements

| Requirement | Notes |
|---|---|
| [Spotube](https://github.com/KRTirtho/spotube) | Version with plugin API v2.0.0 support |
| [Navidrome](https://www.navidrome.org/) | Any recent version |
| ffmpeg *(optional)* | Required for MP3 transcoded streams |

---

## Installation

1. Download `plugin.smplug` from the [Releases](https://github.com/Samuel095383/navi-spotube-plugin/releases) page.
2. In Spotube, open **Settings → Plugins → Install Plugin** and select the downloaded file.
3. The **Navidrome** plugin will appear in the plugin list.  Click **Connect** to open the settings form.

---

## Setup

When you click **Connect** (or **Configure**) on the plugin, a form is shown:

| Field | Example | Description |
|---|---|---|
| Base URL | `http://192.168.1.100:4533` | Full URL to your Navidrome server, **no trailing slash** |
| Username | `admin` | Your Navidrome username |
| Password | `••••••••` | Your Navidrome password |

After you submit the form the plugin automatically pings the server (`/rest/ping.view`).  
If the ping succeeds your credentials are saved and search/playback will work immediately.  
If it fails, an error message is shown and nothing is saved — check your URL and credentials.

---

## Security Notes

> ⚠️ **Always use HTTPS when accessing Navidrome over the internet.**

The plugin authenticates using the Subsonic `enc:` password format, which hex-encodes your password before embedding it in API request query parameters.  
While this avoids plain-text passwords in URLs, it is **not** encryption — anyone who can observe the network traffic can recover the password.

**Recommendations:**

- Access Navidrome only over **HTTPS** (`https://`), especially from outside your home network.
- Use a **reverse proxy** (nginx, Caddy, Traefik) with a valid TLS certificate in front of Navidrome.
- For local network access over plain HTTP (`http://`) the risk is limited to your local network, but HTTPS is still best practice.
- Use a **strong, unique password** for your Navidrome account.

---

## How It Works

### Authentication
Credentials are saved to Spotube's local storage after a successful ping.  
On subsequent app starts the plugin restores its state from local storage without showing the form again.

### Track Search (`matches`)
When Spotube asks the plugin to find audio for a track, the plugin calls:
```
GET /rest/search3.view?query=<title+artist>&songCount=5&...
```
Up to 5 matching songs are returned and mapped to Spotube's `SpotubeAudioSourceMatchObject` format.

### Audio Streaming (`streams`)
For each match the plugin provides two direct stream URLs:

| Stream | Endpoint | Notes |
|---|---|---|
| Original (lossless) | `stream.view?id=<id>` | Serves the file as stored — FLAC, MP3, etc. |
| MP3 320 kbps | `stream.view?id=<id>&format=mp3&maxBitRate=320` | Requires ffmpeg in Navidrome |

Spotube picks the stream that best matches the user's quality preference.

### Auth Parameters in Stream URLs
All stream URLs include authentication query parameters:
```
?id=<song_id>&u=<username>&p=enc:<hex_password>&v=1.16.1&c=navi-spotube
```

---

## Building from Source

### Prerequisites
- [Flutter SDK](https://flutter.dev/docs/get-started/install) (stable channel)
- Hetu script dev tools:
  ```bash
  dart pub global activate hetu_script_dev_tools
  ```

### Build
```bash
make
```
Produces `build/plugin.out`.

### Archive
```bash
make archive
```
Produces `build/plugin.smplug` (a ZIP containing `plugin.json`, `plugin.out`, and `logo.png`).

---

## License

MIT
