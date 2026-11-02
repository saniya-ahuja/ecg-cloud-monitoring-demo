# Real-Time ECG Cloud Monitoring – Mini Demo

A small proof-of-concept of a **device-to-cloud pipeline for clinical signals**.
A simulated ECG is streamed in real time over **MQTT**, received by a "cloud" subscriber,
displayed on a live dashboard, logged, and checked for **transmission delay** and **data loss**.

> Built to understand the core ideas behind real-time remote patient monitoring.
> The ECG is simulated (NeuroKit2); no real patient data is used.

![Live dashboard](images/dashboard.png)

## Pipeline

```
Simulated ECG (250 Hz)  →  Device / sender  →  MQTT broker  →  Cloud / receiver  →  Live dashboard + log
                           (0.1 s packets,                     (loss check,
                            seq no. + timestamp)                latency, heart rate)
```

| Component | Role in demo | Real-world equivalent |
|---|---|---|
| `neurokit2.ecg_simulate` | Generates realistic ECG | ECG sensor + ADC |
| Sender thread | Packs and publishes data | Raspberry Pi |
| `broker.hivemq.com` | Message broker | Private / cloud MQTT broker |
| Receiver | Checks, logs, analyses | Cloud server + database |
| Matplotlib plot | Live display | Web dashboard (e.g. Grafana) |

## How it works

- **Packets:** the signal is sent in 0.1 s chunks (25 samples). Each packet carries a
  **sequence number** and a **send timestamp**.
- **Data loss:** detected from gaps in sequence numbers (expected 11, got 13 → 2 lost).
- **Latency:** `receive time − send time` for each packet.
- **Heart rate:** R-peaks detected with `scipy.signal.find_peaks`; HR = 60 / mean R-R interval.
- **Network faults:** `DROP` parameter deliberately drops packets to test loss detection.

## Results (60 s run, 600 packets)

| Metric | Value |
|---|---|
| Packets received | 600 |
| Packets lost | 0 (0.0%) |
| Mean latency | 94 ms |
| 95th percentile latency | 113 ms |
| Heart rate shown | 76 bpm (simulated: 72 bpm) |

![Latency histogram](images/latency.png)

**Observations**
- Most packets arrived in ~90–95 ms; a small tail reached ~240 ms (network jitter).
  For clinical use, worst-case latency matters, not just the mean.
- The HR reading of 76 bpm came from a **false peak** (a T-wave at the window edge
  detected as an R-peak). Excluding it, the R-R interval is ~0.83 s → **72 bpm**,
  matching the simulation. A robust detector (e.g. Pan–Tompkins) would fix this.

## Limitations

- Simulated ECG, not a real sensor.
- Sender and receiver ran on the same machine, so clocks were synchronised; separate
  devices would need NTP.
- Public broker, QoS 0, no encryption — **not suitable for patient data**.

## Next steps

- Real sensor on a Raspberry Pi (with ADC)
- Private broker with TLS + authentication, QoS 1, and local buffering during outages
- Time-series database (e.g. InfluxDB) and web dashboard
- Real-time analytics on the stream, e.g. arrhythmia / anomaly detection

## Run it

Open `ECG_Cloud_Monitoring_Demo.ipynb` in Google Colab and run all cells top to bottom.

```
pip install neurokit2 paho-mqtt numpy scipy matplotlib
```

## Author

Saniya Ahuja — M.Sc. Data Science, DIAT Pune
