# Network Anomaly Detector

A Python tool for detecting malicious activity in network traffic logs through behavioral analysis. Rather than inspecting packet contents, it reasons about connection patterns — which means it works even on encrypted traffic.

It currently detects three classes of activity:

Port scanning — one host probing many ports (reconnaissance)
C2 beaconing — regular, automated check-ins to a single destination (command-and-control)
Data exfiltration — anomalous outbound data volume to a single destination

Detection is based on statistical baselining of normal traffic, with documented false-positive tuning for each detector.
