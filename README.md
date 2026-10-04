# Network Anomaly Detector

A behavioral network intrusion detection system that detects the three core
stages of an attack — **reconnaissance, command-and-control, and data
exfiltration** — by analyzing connection patterns in network flow logs.

Rather than inspecting packet contents, it reasons about *how* hosts
communicate — connection counts, timing, and data volumes. This means it
works on **encrypted traffic**, where content-based inspection goes blind.

Every detector was built, tested, and refined against real traffic:
public labelled datasets and attacks generated in a self-built lab
(nmap/beacon/exfil → captured with tcpdump → processed with Zeek → detected
in pandas).

## Why behavioral detection?

Most network traffic today is encrypted (HTTPS/TLS), so content-based
detection — reading URLs, payloads, credentials — fails on the majority of
real traffic. Behavioral detection doesn't depend on reading content: it
analyzes connection metadata (who talks to whom, how often, how much, with
what timing), all of which survives encryption. This project focuses on that
encryption-resilient approach, which is increasingly how real detection works.

## The three detectors

### 1. Port scan detection — *reconnaissance*
Flags a source contacting an abnormal number of distinct destination ports.

- **On a public dataset (CIC-IDS2017):** detected scans by per-flow features
  (short duration, few packets, few bytes), tuned to **98% detection at a
  3% false-positive rate**, with thresholds set from the feature distributions.
- **On self-captured traffic (Zeek):** detected scans by host fan-out —
  counting distinct destination ports per source.
- **Problem solved:** the first version flagged the scan *target* as well as
  the scanner, because a scan's bidirectional traffic associates the target
  with many ports too. Diagnosed via Zeek's `conn_state` field (the scanner's
  probes are overwhelmingly reset connections, `RSTRH`; the target's are not)
  and filtered to the scanner's signature, isolating the true attacker.

### 2. Beacon detection — *command-and-control*
Flags source→destination pairs with suspiciously regular connection timing —
the metronomic "check-in" pattern of malware phoning home.

- Computes inter-arrival gaps between connections and measures their
  **standard deviation**: a beacon's gaps barely vary (std ~0.02s), while
  normal traffic is irregular.
- **Problem solved:** the target domain was CDN-backed (Cloudflare), resolving
  to multiple IPs. Grouping by raw IP **fragmented** the beacon across
  addresses, inflating its timing variance to ~45s and hiding it among normal
  traffic. Correlating `conn.log` with `dns.log` to group by **destination
  domain** unified the beacon and recovered its true 0.02s regularity.
- **Dual-threshold logic:** flagging requires *both* low timing variance AND
  a sufficient connection count — variance alone false-positives on sparse
  regular traffic; count alone false-positives on high-volume irregular traffic.

### 3. Exfiltration detection — *data theft*
Flags destinations receiving an abnormally large total data volume.

- Sums total bytes per destination; a bulk transfer (~32MB) stood out trivially
  against normal traffic (max ~2.8KB).
- **Problem solved / design decision:** the captured transfer appeared in the
  *response* byte field rather than the expected *outbound* field (capture
  conditions and an echoing endpoint). The detector sums bytes in **both
  directions** to stay direction-agnostic, catching the volume anomaly however
  the bytes flow.

## Pipeline

Generate attack → Capture traffic → Process to logs → Detect
(nmap / beacon / (tcpdump) (Zeek, in Docker) (pandas)
curl upload)




The lab runs on a Linux VM; Zeek runs in a container (version-independent).
Detectors read Zeek's `conn.log` (and `dns.log` for domain correlation).

## Project structure

- `notebooks/` — one notebook per detector (see `01`–`04`)
- `src/` — detection logic
- `data/` — datasets (raw data git-ignored; see `data/README.md`)
- `tests/` — tests

## Limitations and future work

- **Volume thresholds are context-dependent.** Exfil detection uses a fixed
  byte threshold; real deployment needs a per-destination baseline of normal
  volume. Sophisticated exfil also evades volume detection by trickling data
  out slowly — where the timing/beaconing approach becomes relevant.
- **Scan-state signatures vary.** The scanner-vs-target refinement keys on
  `RSTRH`; other scan types and target behaviours produce different states
  (`S0`, `REJ`, …), which a production detector would also consider.
- **Short demo captures.** Beacons were captured over minutes; real beaconing
  detection operates over much longer windows with more check-ins.
