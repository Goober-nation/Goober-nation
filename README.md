# Timofei Ruchkin

Third-year CS student at Innopolis University. I build backend services and run the infrastructure
they sit on, mostly in Python and Go. Currently looking for a **backend or DevOps internship**.

Everything below is in the repositories on this profile. Where a number came from my own use rather
than a logged benchmark, I say so.

---

## What I work on

| Project | What it is | Stack |
|---|---|---|
| [Post-Vinyl](https://github.com/Goober-nation/Post-Vinyl) | Automated self-hosted music library: find, fetch, tag, serve | Python, FastAPI, SQLite, Docker, SSE |
| [brainmaper](https://github.com/Goober-nation/brainmaper) | AI mind-mapping tool, graph-shaped conversations | Go, MongoDB, React, Docker |
| [quick-vpn](https://github.com/Goober-nation/quick-vpn) | One-shot VPN and proxy provisioning for a fresh VPS | Bash, Docker, systemd, Xray, Dante |
| [lazyhapp](https://github.com/Goober-nation/lazyhapp) | Terminal UI for managing VPN connections | Go, Bubbletea |
| [evil-corp-ads](https://github.com/Goober-nation/evil-corp-ads) | Course project on content injection in RAG pipelines | Python, FAISS, Qwen2.5-3B |
| [trimui_as_bluetooth_controller](https://github.com/Goober-nation/trimui_as_bluetooth_controller) | Handheld console turned into a Bluetooth HID gamepad | C, BlueZ, ARM cross-compilation |
| [SNA_web_server](https://github.com/Goober-nation/SNA_web_server) | HTTP server built from Netcat and Bash | Bash, systemd |
| [ViKi](https://github.com/Innopolis-Robotics-Society/ViKi-dev) | Multi-camera motion capture for robotics (team of 10) | Python, MediaPipe, RGB-D |
| [SingleGoldenRetriever](https://github.com/S1norin/SingleGoldenRetriever) | Real-time messaging over a single Kafka topic (team of 7) | Python, FastAPI, Kafka |

---

## How I work with AI coding agents

Post-Vinyl was built with an AI agent under close direction. I never handed the project over to run
on its own, and the process is documented in the repository rather than described after the fact -
`AGENTS.md`, `backlog/` and `agents_memory/` are about 3,000 lines written while the work happened.

The parts I would defend in an interview:

- **The agent cannot mark its own work complete.** Three states: in progress, AI-tested, user-tested.
  Only I move something into the last one.
- **Definition of Done is mechanical**, not a judgment call: pytest, ruff and mypy all pass, and any
  change touching a real service is either tested live or explicitly labelled unverified.
- **Memory is capped on purpose.** Durable facts live in a file with a hard ~50-line limit; anything
  longer gets pruned into topic files read on demand. An earlier scratchpad approach grew to 791
  lines before I killed it, and the reason is written into the rules so it does not come back.
- **The anti-patterns come from real incidents**, not from a style guide. The strongest one is "never
  duplicate state another service already owns" - it exists because a startup routine was loading
  every stored response and made boot time grow with all past history.

---

## Things I debugged

These took longer than the code did, and they are the parts I actually enjoy.

<details>
<summary><b>A single search opening 7,000 sockets - and disproving my own fix</b></summary>

One search against the Soulseek network was opening roughly 6,000 to 7,000 host sockets. The obvious
fix was to cap how many peer responses I asked for, so I tried it and measured.

It made things worse. Asking for 10x fewer responses cost about 5% **more** sockets, because the
setting is local bookkeeping and does not reduce the connections actually opened. I kept it only
because it is harmless for capping processed results, and solved the real problem with a query cache,
a sliding-window rate limiter (4 searches per 60 seconds, 429 past that) and 2-second pacing per
track.
</details>

<details>
<summary><b>A geo-block that was really a DNS privacy leak</b></summary>

One container kept getting a geo-blocked response from an upstream API. It looked like a VPN or
DNS-filter problem.

It was EDNS Client Subnet. The provider's authoritative DNS returns 127.0.0.1 to any resolver that
forwards client-subnet data along with the query. I confirmed it by testing resolvers that forward
ECS (Cloudflare, Google, Yandex, AdGuard, OpenDNS - all affected) against ones that strip it (Quad9,
Verisign - all clean), and ruled out local DoH interception separately. The fix was to scope a custom
DNS block to the single affected container rather than change resolvers everywhere.

A follow-on: posters still loaded inconsistently across devices, because image caching was off and
each browser was resolving the image host itself. A server-side DNS fix cannot help with that.
</details>

<details>
<summary><b>Indexer results that were rejected for the wrong reason</b></summary>

Valid releases were being rejected and it looked like a quality-profile problem. Setting the profile
to accept anything did not help, which ruled that out - the real cause was title parsing.

Two separate cases fell out of that. Anime releases collapsed to "Unknown Series" because that
sub-forum has no `SxxExx` token in its titles; I distinguished this from a competing "per-episode
search does not accept packs" theory by testing the API directly. And a movie search returned nothing
because the search used the TMDB release year while the tracker tags the production year, and the
indexer's Torznab only does literal substring matching.

I also spent time on approaches that did not work and did not retry them: the indexer's title-rewrite
toggles mangled titles rather than fixing them, and a custom Cardigann override turned out to be
impossible because that indexer is a compiled C# implementation, not YAML-defined.
</details>

<details>
<summary><b>A blank panel caused by a missing resize event</b></summary>

An embedded qBittorrent UI rendered blank inside a dashboard iframe. I read the dashboard's own
`functions.js` and found it hides and shows tabs without firing a resize event, while the embedded UI
virtualises its table based on container height - so it measured zero rows. I switched that tab to a
popout type instead of patching someone else's application.
</details>

---

## Testing

The part of Post-Vinyl I am most pleased with is `tests/live/`, an opt-in harness that drives the
real Docker stack instead of mocks.

It has to deal with the fact that results are not deterministic - which peers answer depends on who
is online right now. So:

- The track corpus is **stratified by failure mode**, not by popularity: `POPULAR` titles (if these
  fail, the bug is mine), `AWKWARD` ones (feat. clauses, parentheses, non-ASCII, long titles, which
  stress query building and path sanitisation) and `RARE` ones (thin seeding, which stress retry and
  alternative-peer selection).
- **Run order is deliberately the opposite of reporting weight.** Easy cases run first so a broken
  pipeline is obvious immediately; rare failures rank first in the report, because they say more.
- The pass-ratio collector **reads the spread across repeated runs** rather than trusting one number,
  because an early cutoff caps the sample at about 10 peers regardless of how popular a track is.
- The harness detects a **stale container image** and tells you to rebuild, instead of surfacing a
  missing-migration error that reads like a harness bug.

Alongside that: 1,193 test functions across unit, route, worker and live tiers.

---

## Infrastructure

I run my own hardware and expose parts of it publicly without exposing the network behind it.

- **Nginx on a cheap VPS forwards traffic** - including TLS challenge traffic - to home hardware over
  a private Tailscale tunnel. Subdomain per service, wildcard certificate, websocket support.
- **Wildcard TLS through Let's Encrypt DNS-01**, using `certbot-regru` against a registrar with no
  native ACME support, after a few manual attempts through its admin panel.
- **coturn** (STUN/TURN, systemd) and a **Tailscale DERP relay**, both configured by me, for
  self-hosted NAT traversal.
- A **media automation stack** I composed: GPU passthrough for transcoding, YAML anchors for shared
  configuration, and an HTTP-to-SOCKS5 sidecar applied to only the services that need it. Migrated
  the whole host from macOS to Ubuntu with a configuration that turned out to be portable across
  architectures.

I also **run** (deployed and operate, did not build or architect) Immich, a Revolt-compatible chat
platform, Piwigo and Dockhand.

---

## A note on two projects

**evil-corp-ads** was built for a deep-learning-for-search course. It studies how injected commercial
content can hijack the output of a retrieval-augmented pipeline: a summarizer, an injector and a
reviewer chained together, with a separate LLM judge scoring how far the injection got. The repository
name and tagline are jokes made for a course, and I would rather say that plainly than dress it up.

The part I would actually point at is the evaluation harness - nDCG@10 with graded relevance,
precision@10, recall@10 and MRR implemented from scratch and run against a BEIR dataset, plus a
comparison of an FP32 FAISS index against an INT8-quantized one on latency and process memory.

**ViKi** is a summer capstone with ten contributors, and I want to be precise about which parts are
mine. I owned the capture recorder, the skeleton and robot visualization, the dataset-export API
routes, trajectory preparation and the server worker layer. I originated the hand-tracking approach -
subtracting a captured static depth map to lift the hand out of a cluttered scene, then taking the
median depth over a region of interest with outlier rejection - and both mechanisms are still in the
codebase, though a teammate later rewrote the modules around them. The inverse-kinematics retargeting
core is not my work. The accuracy improvement the project reports was measured with a teammate's
evaluation tooling, so I would describe it as a project result rather than my own measurement.

---

## Some numbers, and where they come from

- **1,193** test functions in Post-Vinyl, and 11 SQLite migrations. Countable in the repository.
- **Usable search results went from roughly 10% to 70-80%** after I replaced plain "song + artist"
  matching with a query builder and result filter. This is from my own use over months, not a logged
  benchmark - the query builder is tuned on a measurable finding though: two-word queries return
  results reliably on that network, while three-or-more-word combined queries tend to return nothing.
- **About 26 of 30 test cases** returned 250+ peer responses in under 10 seconds.
- Roughly **3,600 surviving lines** of the ViKi codebase are mine, second-largest of ten contributors.

---

## Stack

**Comfortable:** Python, FastAPI, Docker and Docker Compose, Bash, Linux administration, Git

**Working knowledge:** Go (standard-library `net/http`, Bubbletea), Nginx and TLS, networking
(DNS, proxying, NAT/STUN/TURN, VPN protocols), systemd, SQLite and schema migrations, Kafka,
MongoDB, pytest / ruff / mypy

**Have used:** C and ARM cross-compilation, FAISS and vector search, React, MediaPipe and RGB-D capture

---

## Education

**Innopolis University** - B.Sc. Computer Science, third year of four. GPA 4.4/5.
Transferred from a grant place to a fully government-funded place.

Coursework worth naming: Distributed and Network Programming, System and Network Administration,
Data Modeling and Databases II, Operating Systems, Networks.

---

Russian (native), English (fluent). Reachable through the email on my profile.
