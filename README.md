# Cat TV for Raspberry Pi

An automated Cat TV system that plays YouTube bird, squirrel, fish, and other cat-entertainment videos on a television according to a configurable daily schedule.

This version is based on Cat TV 2.0 and adds support for current YouTube playback using yt-dlp, Deno, yt-dlp-ejs, separate video/audio streams, MPV, direct DRM/KMS video output, and ALSA HDMI audio. The original project provides the scheduling, web interface, fallback videos, playback history, and systemd service architecture. [github.com]

## Features
- Automatic scheduled Cat TV playback
- Web-based control interface
- Automatic YouTube search
- Multiple fallback videos
- Current yt-dlp YouTube extraction
- Deno JavaScript challenge support
- Separate YouTube video and audio stream handling
- MPV playback
- Direct DRM/KMS fullscreen HDMI output
- Direct ALSA HDMI audio
- Raspberry Pi 3-friendly H.264 video selection
- Runs without a graphical desktop
- Automatic startup using systemd
- Stream URLs redacted from application logs

## Tested Hardware
This configuration has been tested on:
- Raspberry Pi 3 Model B Rev 1.2
- HDMI television
- Raspberry Pi OS Lite / Debian Trixie
- 64-bit ARM (aarch64)
A wired Ethernet connection is recommended for long-duration YouTube streaming, although Wi-Fi can also be used.

## Installation
### 1. Install Raspberry Pi OS Lite
Use Raspberry Pi Imager to install Raspberry Pi OS Lite onto a microSD card. Raspberry Pi recommends Imager for installing Raspberry Pi OS, whose current generation is based on Debian Trixie. [raspberrypi.com]  
During imaging, configure:
- hostname, for example raspberrypi
- username and password
- Wi-Fi, if required
- SSH access
- locale and timezone
SSH provides remote terminal access to the Raspberry Pi over your local network. [raspberrypi.com]  
Connect the television to the Pi using HDMI.  
Boot the Raspberry Pi.

### 2. Connect using SSH
Find the Raspberry Pi IP address from your router or locally with:
```shell
hostname -I
```
Then connect from another computer:
```shell
ssh YOUR-USERNAME@RASPBERRY-PI-IP
```

### 3. Update Raspberry Pi OS
```shell
sudo apt update
sudo apt full-upgrade -y
```
Reboot if required:
```shell
sudo reboot
```
Reconnect using SSH afterward.

### 4. Install required system packages
Install Git, curl, FFmpeg, MPV, ALSA utilities, and supporting packages:
```shell
sudo apt update
sudo apt install -y /
  git /
  curl /
  unzip /
  ffmpeg /
  mpv /
  alsa-utils /
  python3 /
  python3-venv
```
Verify:
```shell
ffmpeg -version
mpv --version
aplay --version
```

### 5. Install uv
Install the Python uv package manager:
```shell
curl -LsSf https://astral.sh/uv/install.sh | sh
```
Reload the shell:
```shell
source ~/.bashrc
```
Verify:
```shell
uv --version
```

### 6. Clone Cat TV
```shell
cd ~
git clone https://github.com/grove8993/cat_tv.git
cd cat_tv
```

### 7. Install Python dependencies
```shell
uv sync
```
Verify yt-dlp:
```shell
uv run yt-dlp --version
```

## YouTube support
Modern YouTube extraction requires JavaScript challenge processing. This project uses:
- yt-dlp
- yt-dlp-ejs
- Deno
The current yt-dlp EJS architecture supports Deno as the recommended JavaScript runtime.

### 8. Install Deno
Install Deno using the official Linux installer:
```shell
curl -fsSL https://deno.land/install.sh | sh
```
Add it to the current shell:
```shell
export PATH="$HOME/.deno/bin:$PATH"
```
Verify:
```shell
deno --version
```
Deno supports Linux ARM64 and the official documentation recommends its shell installer for current Linux installations. [docs.deno.com]

### 9. Verify EJS support
From the Cat TV directory:
```shell
uv run python -c "import yt_dlp_ejs; print('yt-dlp-ejs installed')"
```
Expected:
```plain text
yt-dlp-ejs installed
```
Then verify yt-dlp detects Deno:
```shell
uv run yt-dlp -v -g "https://www.youtube.com/watch?v=VIDEO_ID"
```
Near the beginning of the output, look for:
```plain text
yt_dlp_ejs
JS runtimes: deno
JS Challenge Providers: ... deno
```

## Cat TV configuration
### 10. Create .env
```shell
cp .env.example .env
nano .env
```
At minimum, configure:
```shell
PLAYER_BACKEND=mpv
AUDIO_OUTPUT=hdmi
```
The web interface defaults to port 5000.  
A YouTube API key is optional but can improve search functionality.

### 11. Verify the HDMI audio device
Run:
```shell
aplay -l
```
On the tested Raspberry Pi 3 configuration, HDMI appeared as:
```plain text
card 1: vc4hdmi
device 0: MAI PCM i2s-hifi-0
```
Test HDMI audio:
```shell
speaker-test -D hw:1,0 -c 2 -t sine
```
Stop with Ctrl+C.  
If your HDMI device is not 1,0, adjust the MPV ALSA device configuration accordingly.

## Raspberry Pi 3 video configuration
The Raspberry Pi 3 does not handle modern high-resolution AV1 YouTube formats well.  
This project therefore prefers:
- H.264 / AVC
- 720p or lower
- 30 FPS or lower
The yt-dlp format selector used by this version is designed around:
```plain text
bestvideo[vcodec^=avc1][height<=720][fps<=30]+bestaudio
```
This avoids formats such as YouTube AV1 format 401, which produced severely incorrect video rendering on the tested Raspberry Pi 3.

## Install the systemd service
### 12. Run the application's installer/service setup
```shell
chmod +x install.sh
./install.sh
```
The original Cat TV project provides automated installation and systemd operation. [github.com]  
Ensure the service can find Deno.  
Run:
```shell
sudo systemctl edit cat-tv
```
Add:
```ini
[Service]
Environment=DISPLAY=
Environment="PATH=/home/YOUR-USERNAME/.deno/bin:/home/YOUR-USERNAME/.local/bin:/usr/local/bin:/usr/bin:/bin"
SupplementaryGroups=video render audio
```
Replace YOUR-USERNAME with the Linux account running Cat TV.  
Then:
```shell
sudo systemctl daemon-reload
sudo systemctl enable cat-tv
sudo systemctl restart cat-tv
```

### 13. Verify Cat TV
```shell
sudo systemctl status cat-tv --no-pager
```
Expected:
```plain text
Active: active (running)
```
Watch logs:
```shell
journalctl -u cat-tv -f
```

## Web interface
Determine the Raspberry Pi's address:
```shell
hostname -I
```
Then open:
```plain text
http://RASPBERRY-PI-IP:5000
```
For example:
```plain text
http://192.168.1.100:5000
```
The original Cat TV web application provides Dashboard, Channels, Schedules, and History functionality. [github.com]

## Configure the schedule
Use the web interface to define when Cat TV should play.  
The original application's default schedule is:
- Morning: 7:00 AM - 11:00 AM
- Evening: 5:00 PM - 8:00 PM
These times can be changed from the web interface. [github.com]  
Verify the Pi timezone:
```shell
timedatectl
```
Change it if necessary, for example:
```shell
sudo timedatectl set-timezone America/Chicago
```

## Troubleshooting
### Check the service
```shell
sudo systemctl status cat-tv --no-pager -l
```
### Follow logs
```shell
journalctl -u cat-tv -f
```
### Check port 5000
```shell
sudo ss -ltnp | grep :5000
```
### Test the web server locally
```shell
curl -I http://localhost:5000
```
### Check HDMI audio
```shell
aplay -l
```
### Check DRM devices
```shell
ls -l /dev/dri/
```
The Cat TV account should have video, render, and audio access:
```shell
groups
```

## Important notes for Raspberry Pi 3
Avoid allowing yt-dlp to automatically choose the highest-resolution video.  
Formats such as:
- 4K
- 60 FPS
- AV1
- HDR
are inappropriate targets for a Raspberry Pi 3.  
For this hardware, the recommended playback target is:
- H.264
- ≤720p
- ≤30 FPS
This provides much more reliable decoding and correct color reproduction.

## Security and logging
Temporary Google/YouTube playback URLs are signed URLs and should not be written verbatim to persistent logs.  
This version redacts player URLs and instead logs entries such as:
- Selected video format XXX + audio format XXX
- Running player command: mpv ... --audio-file=[REDACTED] [STREAM-URL-REDACTED]

## Updating
Before updating the project, take a working backup:
```shell
cd ~
cp -a cat_tv cat_tv.backup
```
Then inspect upstream changes before merging them.  
Because YouTube and yt-dlp behavior changes over time, upgrades to yt-dlp, yt-dlp-ejs, Deno, MPV, or the player selection logic should be tested before deploying them to an unattended Cat TV.

## Credits
Based on the original Cat TV 2.0 project by Jeff Crouse. [github.com]  
This fork adds changes needed for current YouTube playback and Raspberry Pi 3 compatibility, including Deno/EJS challenge solving, MPV, separate audio/video streams, H.264 selection, direct HDMI audio, and safer stream logging.  
The original project is distributed under the MIT License. [github.com]
