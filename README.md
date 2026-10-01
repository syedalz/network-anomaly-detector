# Network Anomaly Detector

> **Work in progress** — actively building. See [Roadmap](#roadmap) for status.

A Python tool for detecting malicious activity in network traffic logs through
behavioral analysis. Rather than inspecting packet contents, it reasons about
connection *patterns* — which means it works even on encrypted traffic, where
content-based inspection goes blind.

It is designed to detect three classes of activity, each mapped to a stage of a
real intrusion:

- **Port scanning** — one host probing many ports in a short window (reconnaissance)
- **C2 beaconing** — regular, automated check-ins to a single destination (command-and-control)
- **Data exfiltration** — anomalous outbound data volume to a single destination

Detection is based on statistical baselining of normal traffic, with documented
false-positive tuning for each detector.

## Why behavioral detection?

Most network traffic today is encrypted (HTTPS/TLS), so you often can't read what's
inside the packets. Content-based detection fails on encrypted traffic; behavioral
detection does not, because it analyzes *how* hosts communicate — connection counts,
timing, volumes, destinations — rather than *what* they send. This project focuses
on that encryption-resilient approach.

## How it works

The tool ingests network connection logs (Zeek `conn.log` and related logs),
loads them into pandas, and applies statistical detectors:

| Detector      | Signature it looks for                                              |
|---------------|--------------------------------------------------------------------|
| Port scan     | One source contacting many distinct ports, with many failed connections |
| Beaconing     | Connection pairs with suspiciously regular timing (low variance)   |
| Exfiltration  | Outbound transfers in the top percentile of data volume            |

Each detector is tuned against labelled data, and its false-positive behavior is
documented rather than hidden.

## Project Structure

- `src/` — detection logic and log parsing
- `data/` — datasets (raw data not committed; see `data/raw/`)
- `notebooks/` — exploratory analysis
- `tests/` — unit tests

## Getting Started

```bash
# Clone the repo
git clone https://github.com/syedalz/network-anomaly-detector.git
cd network-anomaly-detector

# Set up the environment
python -m venv venv
source venv/bin/activate      # Windows: .\venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

Raw datasets are not committed to the repo (they're large). See `data/raw/` for
where to place downloaded data.

## Roadmap

- [x] Project setup and environment
- [ ] Load and parse Zeek `conn.log` into pandas
- [ ] Exploratory analysis of normal vs. malicious traffic
- [ ] Port scan detector
- [ ] Beaconing detector
- [ ] Exfiltration detector
- [ ] False-positive tuning and evaluation
- [ ] Tests
- [ ] Writeup / blog post

## About

Built as a hands-on project in network security monitoring and detection
engineering, focusing on the behavioral analysis techniques used in SOC and
threat-detection work.
