# Data

Raw data is **not committed** to this repository (it's large and/or
regenerable). This file explains where each dataset comes from and how to
obtain or reproduce it. Small samples for running the detectors live in
`samples/`.

## Sources

### CIC-IDS2017 (downloaded)
Public intrusion-detection dataset of labelled network flows.

- **Used for:** the port-scan detector (per-flow feature analysis).
- **File used:** `Friday-WorkingHours-Afternoon-PortScan.pcap_ISCX.csv`
  (the MachineLearningCSV release).
- **Download:** https://www.unb.ca/cic/datasets/ids-2017.html
  (fill in the short request form to get the download link).
- **Place in:** `raw/cicids2017/`

### Zeek captures (self-generated)
Network logs generated in a local lab to demonstrate the full
capture-to-detection pipeline on traffic I produced myself.

- **Used for:** running the detector on real, self-captured attack traffic.
- **How it was generated:**
  1. nmap scan run against the authorised practice host `scanme.nmap.org`
     from an isolated lab VM.
  2. Traffic captured to a pcap with `tcpdump`.
  3. pcap processed into Zeek logs (`conn.log`) using the official Zeek
     Docker image.
- **Place in:** `raw/zeek-captures/`

## Folder layout

- `raw/` — raw datasets (git-ignored; obtain per the instructions above)
- `samples/` — small committed excerpts so the detectors can be run without
  the full datasets