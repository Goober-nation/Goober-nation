<h1 align="center">Timofei Ruchkin</h1>

<p align="center">
Third-year CS student at Innopolis. I build backend services and run the infrastructure under them.<br>
Looking for a <b>backend or DevOps internship</b>.
</p>

<p align="center">
<img src="https://img.shields.io/badge/Go-00ADD8?style=for-the-badge&logo=go&logoColor=white" alt="Go">
<img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python">
<img src="https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white" alt="FastAPI">
<img src="https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white" alt="Docker">
<img src="https://img.shields.io/badge/Linux-000000?style=for-the-badge&logo=linux&logoColor=white" alt="Linux">
<img src="https://img.shields.io/badge/Nginx-009639?style=for-the-badge&logo=nginx&logoColor=white" alt="Nginx">
<img src="https://img.shields.io/badge/Kafka-231F20?style=for-the-badge&logo=apachekafka&logoColor=white" alt="Kafka">
<img src="https://img.shields.io/badge/MongoDB-47A248?style=for-the-badge&logo=mongodb&logoColor=white" alt="MongoDB">
</p>

---

### The setup I actually run

A public domain served from home hardware, without opening the home network to it: nginx on the
cheapest VPS tier forwards traffic, ACME challenges included, to a home server over a private
Tailscale tunnel. Wildcard certificates come from Let's Encrypt DNS-01 against a registrar with no
ACME support. Post-Vinyl, a GPU-transcoding media stack, coturn and a Tailscale DERP relay all sit
behind it. Nothing at home is exposed directly.

---

### Projects

| | | |
|---|---|---|
| [**Post-Vinyl**](https://github.com/Goober-nation/Post-Vinyl) | Automated self-hosted music library: find, fetch, tag, serve | `Python` `FastAPI` `SQLite` `Docker` |
| [**brainmaper**](https://github.com/Goober-nation/brainmaper) | AI mind-mapping, graph-shaped conversations | `Go` `MongoDB` `React` |
| [**quick-vpn**](https://github.com/Goober-nation/quick-vpn) | One-shot VPN + proxy provisioning for a fresh VPS | `Bash` `Xray` `systemd` |
| [**lazyhapp**](https://github.com/Goober-nation/lazyhapp) | Terminal UI for managing VPN connections | `Go` `Bubbletea` |
| [**evil-corp-ads**](https://github.com/Goober-nation/evil-corp-ads) | Course project on content injection in RAG pipelines | `FAISS` `Qwen2.5-3B` |
| [**trimui...controller**](https://github.com/Goober-nation/trimui_as_bluetooth_controller) | Handheld console turned into a Bluetooth gamepad | `C` `BlueZ` `ARM` |

Team work: [ViKi](https://github.com/Innopolis-Robotics-Society/ViKi-dev) (motion capture, 10 people)

and [SingleGoldenRetriever](https://github.com/S1norin/SingleGoldenRetriever) (Kafka messaging, 7 people).

---

### How I work

- **Try unconventional solutions.** Can't knock it 'till you try it, it may turn into innovation
- **Test against real services.** built a harness that drives the actual Docker stack
- **AI agents work under a protocol, never autonomously.** Three states, and only I can mark something complete

---

**Innopolis University**, B.Sc. Computer Science, 3rd year. GPA 4.4/5.

📫 **goobernation@duck.com**
