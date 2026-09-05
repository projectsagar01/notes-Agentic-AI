---
{"dg-publish":true,"permalink":"/src/site/notes/notes/Agentic Ai/Public-Notes/Roadmap/obs-setup/","dg-note-properties":{}}
---

# Master Open-Source Multistreaming Setup Guide

This master guide provides the complete end-to-end instructions for deploying a 100% free, open-source local multistreaming server on Linux (Podman + NGINX-RTMP + Stunnel) to broadcast simultaneously to **YouTube** and **Kick**, optimized with OBS Studio and professional audio engineering.

---

## Phase 1: NGINX RTMP Server Configuration

1. **Create the Configuration File:**
Open a terminal and create the NGINX configuration file in your home directory:
```bash
nano ~/multistream.conf

```


2. **Add the Routing Logic:**
Paste the following block into the file. Replace `YOUR_YOUTUBE_STREAM_KEY` with your actual YouTube stream key:Haan bhai, bilkul sahi socha tune! Jab tu laptop restart karta hai, toh RAM aur background processes khali ho jaate hain. Isiliye tera NGINX container aur Stunnel band ho chuke hain.

OBS jab `127.0.0.1:1935` par connection bhej raha hai, toh abhi wahan darwaza kholne ke liye koi server zinda nahi hai—isiliye **"Failed to connect"** aa raha hai.

Har baar laptop restart karne ke baad, stream shuru karne se pehle tujhe apne local server ko **wapas on karna padega**. Isey start karne ke liye bas yeh 2 simple steps karne hain:

### Step 1: NGINX Container Zinda Kar (Terminal 1)

Terminal khol aur pehle se bane hue container ko start karne ke liye yeh command chala:

```bash
podman start my-stream-server

```

### Step 2: Stunnel On Kar (Terminal 2)

Ek doosra terminal tab khol (ya naya window) aur Kick ke encryption ke liye stunnel chala do:

```bash
stunnel ~/kick-stunnel.conf

```

*(Is terminal ko aise hi khula rehne dena jab tak tu streaming kar raha hai).*

---

### Uske Baad:

Jaise hi yeh dono chal jayein, tu **OBS khol kar "Start Streaming" daba dena**. Server background mein active ho chuka hoga, aur tera YouTube + Kick ka multistreaming setup ekdum makkhan ki tarah chalne lagega!
```nginx
worker_processes auto;
rtmp_auto_push on;
events {}

rtmp {
    server {
        listen 1935;
        chunk_size 4096;

        application live {
            live on;
            record off;

            # Push plain RTMP directly to YouTube
            push rtmp://a.rtmp.youtube.com/live2/YOUR_YOUTUBE_STREAM_KEY;

            # Push locally to Stunnel for Kick's secure RTMPS requirement
            push rtmp://127.0.0.1:1936/app/YOUR_KICK_STREAM_KEY;
        }
    }
}

```


*Save and exit:* Press `Ctrl + O`, hit `Enter`, then press `Ctrl + X`.

---

## Phase 2: SSL Encryption Tunnel for Kick (Stunnel)

Because Kick enforces secure RTMPS (AWS IVS infrastructure) while standard NGINX-RTMP modules push plain RTMP, an SSL tunnel bridges the connection locally.

1. **Install Stunnel:**
```bash
sudo dnf install stunnel

```


2. **Create the Stunnel Configuration:**
Create a configuration file named `~/kick-stunnel.conf`:
```bash
nano ~/kick-stunnel.conf

```


3. **Paste the Tunnel Parameters:**
```ini
foreground = yes
client = yes

[kick-rtmps]
accept = 127.0.0.1:1936
connect = fa723fc1b171.global-contribute.live-video.net:443

```


*Save and exit:* Press `Ctrl + O`, hit `Enter`, then press `Ctrl + X`.

---

## Phase 3: Launching the Backend Services

1. **Start the NGINX Podman Container:**
Remove any existing containers and launch the fresh server instance with host network permissions:
```bash
podman rm -f my-stream-server
podman run -d --network host --name my-stream-server -v ~/multistream.conf:/etc/nginx/nginx.conf:z docker.io/tiangolo/nginx-rtmp

```


2. **Run the SSL Tunnel:**
In a separate terminal tab, start the stunnel process to keep encryption active:
```bash
stunnel ~/kick-stunnel.conf

```



---

## Phase 4: OBS Studio & Keyframe Optimization

1. **Configure Stream Destination:**
* Open OBS Studio and navigate to **Settings -> Stream**.
* Set **Service** to **Custom...**.
* Set **Server URL** to: `rtmp://127.0.0.1/live`
* Set **Stream Key** to any placeholder string (e.g., `test`), since your actual keys are handled securely inside the NGINX configuration file.


2. **Configure Keyframe Interval:**
* Go to **Settings -> Output**.
* Change **Output Mode** from Simple to **Advanced**.
* Locate **Keyframe Interval** and set it strictly to **`2`** seconds to satisfy platform latency requirements.



---

## Phase 5: Professional Audio Filters & Echo Prevention

1. **Apply Microphone Filters:**
Click the three dots/gear icon next to your microphone in the OBS Audio Mixer, select **Filters**, and add the following filters **in this exact sequence**:
* **Noise Suppression:** Set the method to **RNNoise** (AI-powered background static and fan noise cleaner).
* **Gain:** Adjust upward (e.g., `+5.00 dB` to `+8.00 dB`) until normal speech peaks in the yellow meter zone.
* **Limiter:** Set the threshold to **-3.00 dB** to prevent audio clipping during loud moments.


2. **Eliminate Audio Echo:**
* Mute or close any browser tabs where your live stream preview is playing with unmuted audio to prevent feedback loops.
* Ensure audio monitoring is set to **Monitor Off** under *Advanced Audio Properties* in OBS.


Haan bhai, bilkul sahi socha tune! Jab tu laptop restart karta hai, toh RAM aur background processes khali ho jaate hain. Isiliye tera NGINX container aur Stunnel band ho chuke hain.

  

OBS jab `127.0.0.1:1935` par connection bhej raha hai, toh abhi wahan darwaza kholne ke liye koi server zinda nahi hai—isiliye **"Failed to connect"** aa raha hai.

  

Har baar laptop restart karne ke baad, stream shuru karne se pehle tujhe apne local server ko **wapas on karna padega**. Isey start karne ke liye bas yeh 2 simple steps karne hain:

  

### Step 1: NGINX Container Zinda Kar (Terminal 1)

Terminal khol aur pehle se bane hue container ko start karne ke liye yeh command chala:

  

Bash

```
podman start my-stream-server
```

### Step 2: Stunnel On Kar (Terminal 2)

Ek doosra terminal tab khol (ya naya window) aur Kick ke encryption ke liye stunnel chala do:

  

Bash

```
stunnel ~/kick-stunnel.conf
```

_(Is terminal ko aise hi khula rehne dena jab tak tu streaming kar raha hai)._

  

### Uske Baad:

Jaise hi yeh dono chal jayein, tu **OBS khol kar "Start Streaming" daba dena**. Server background mein active ho chuka hoga, aur tera YouTube + Kick ka multistreaming setup ekdum makkhan ki tarah chalne lagega!