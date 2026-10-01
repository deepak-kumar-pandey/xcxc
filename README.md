# AI-Powered Network Intrusion Detection System (NIDS)

An intelligent, multi-class Network Intrusion Detection System built using Machine Learning to identify, classify, and mitigate cyber threats across enterprise networks.

The system pairs a high-performance **Random Forest Classifier** trained on the benchmark **CICIDS2017** dataset with a real-time **Scapy** packet capture pipeline and an interactive **Streamlit** threat intelligence dashboard.

---

## 📌 Executive Summary

Traditional rule-based intrusion detection systems (such as legacy Snort or Suricata rule sets) struggle to identify zero-day vulnerabilities, polymorphism, and subtle low-and-slow volumetric anomalies.

This project demonstrates an automated, data-driven approach by transforming raw packet captures into bidirectional statistical flows and running inference against a trained ensemble model.

### Detection Scope

Multi-class classification across **7 distinct traffic profiles**:

- Normal Traffic
- DoS
- DDoS
- Port Scanning
- Web Attacks
- Brute Force
- Botnets

### Model Benchmark

Evaluated on the **Canadian Institute for Cybersecurity IDS 2017 (CICIDS2017)** dataset (~2.8M records), achieving approximately **99.90% classification accuracy**.

### Dual Operation Modes

1. **Batch Traffic Analysis**
   - Enterprise-grade log auditor
   - Parses offline flow records
   - Provides visual threat distribution telemetry

2. **Live Packet Ingestion**
   - Hardware-interfacing network sniffer
   - Uses Scapy and Npcap
   - Aggregates raw Ethernet frames into stateful flows dynamically

---

## 🏗️ System Architecture

```text
                       ┌───────────────────────────┐
                       │   Network Traffic Sources  │
                       └─────────────┬─────────────┘
                                     │
                    ┌────────────────┴────────────────┐
                    │                                 │
          ┌─────────▼─────────┐             ┌────────▼─────────┐
          │ Live Network      │             │ Batch PCAP /     │
          │ Interface         │             │ CSV Logs         │
          └─────────┬─────────┘             └────────┬─────────┘
                    │                                 │
              (Raw Packets)                           │
                    │                                 │
          ┌─────────▼─────────┐                       │
          │ Scapy Packet      │                       │
          │ Sniffer           │                       │
          └─────────┬─────────┘                       │
                    │                                 │
          ┌─────────▼─────────┐                       │
          │ Session Flow      │                       │
          │ Tracker           │                       │
          │ (Duration, IAT,   │                       │
          │ Size)             │                       │
          └─────────┬─────────┘                       │
                    └────────────────┬────────────────┘
                                     │
                           ┌─────────▼─────────┐
                           │ Feature Engineering│
                           │ & Alignment         │
                           │ (52 Flow Features  │
                           │ + Normalization)    │
                           └─────────┬─────────┘
                                     │
                           ┌─────────▼─────────┐
                           │ Robust Scaler     │
                           │ (Standardization) │
                           └─────────┬─────────┘
                                     │
                           ┌─────────▼─────────┐
                           │ Random Forest     │
                           │ Classifier        │
                           │ (50 Decision      │
                           │ Trees)            │
                           └─────────┬─────────┘
                                     │
                    ┌────────────────┴────────────────┐
                    │                                 │
          ┌─────────▼──────────┐            ┌────────▼───────────┐
          │ Real-Time Terminal │            │ Streamlit Telemetry│
          │ Alert              │            │ Web UI              │
          │                    │            │                     │
          │ - Flow 5-Tuple     │            │ - Threat Distribution│
          │ - Attack Class     │            │   (Donut Chart)     │
          │ - Probability      │            │ - Normal vs         │
          │   Confidence       │            │   Malicious KPI     │
          │                    │            │   Cards             │
          │                    │            │ - Dynamic Attack    │
          │                    │            │   Signature Tester  │
          └────────────────────┘            └─────────────────────┘
```

---

## 📊 Dataset & Feature Engineering

The model is trained on the **CICIDS2017** benchmark, reflecting modern network attacks, realistic background traffic including HTTP, HTTPS, FTP, SSH, and Email, and deep flow metadata generated via **CICFlowMeter**.

### Monitored Traffic Categories

| Category | Attack Sub-Types Included in Benchmark |
|---|---|
| Normal Traffic | Benign daily enterprise traffic (Web, Mail, Cloud sync) |
| DoS | DoS Hulk, DoS GoldenEye, DoS Slowloris, DoS Slowhttptest |
| DDoS | High-volume LOIC UDP/TCP synchronized flood |
| Port Scanning | Nmap stealth sweeps, SYN scans, OS fingerprinting |
| Brute Force | SSH-Patator, FTP-Patator credential stuffing |
| Web Attacks | Cross-Site Scripting (XSS), SQL Injection, Command Injection |
| Bots | Ares Botnet command-and-control (C2) communications |

### Key Flow Features Monitored

The model uses **52 flow parameters**.

#### Temporal Dynamics

- `Flow Duration`
- `Flow IAT Mean`
- `Flow IAT Std`
- `Flow IAT Max`
- `Flow IAT Min`
- `Fwd IAT Total`
- `Bwd IAT Mean`

#### Volumetric Metrics

- `Total Fwd Packets`
- `Total Backward Packets`
- `Total Length of Fwd Packets`
- `Total Length of Bwd Packets`

#### Packet Size Statistics

- `Fwd Packet Length Max`
- `Fwd Packet Length Min`
- `Fwd Packet Length Mean`
- `Packet Length Std`
- `Packet Length Variance`

#### TCP Control Semantics

- `FIN Flag Count`
- `SYN Flag Count`
- `RST Flag Count`
- `PSH Flag Count`
- `ACK Flag Count`
- `URG Flag Count`

#### Window & Subflow Attributes

- `Init_Win_bytes_forward`
- `Init_Win_bytes_backward`
- `Subflow Fwd Packets`
- `Subflow Fwd Bytes`

---

## 📁 Repository Structure

```text
AI_Intrusion_Detection/
│
├── data/
│   ├── raw/
│   │   ├── cicids2017_cleaned.csv      # Preprocessed benchmark training dataset
│   │   └── demo_sample_traffic.csv     # Balanced 700-row dataset for dashboard evaluation
│   │
├── models/
│   ├── random_forest_model.pkl         # Serialized Random Forest classifier
│   ├── scaler.pkl                      # Scikit-learn feature scaler
│   ├── label_encoder.pkl               # Target class label encoder
│   └── feature_names.pkl               # 52 verified feature column mapping
│
├── src/
│   ├── app.py                          # Streamlit telemetry web application
│   ├── live_detector.py                # Real-time packet sniffer & flow assembler
│   ├── test_attack_detection.py        # Ground-truth signature verification harness
│   └── simulate_dos.py                 # Socket-based UDP flood testing utility
│
├── requirements.txt                    # Environment package dependencies
└── README.md                           # Engineering & operational documentation
```

---

## ⚙️ Installation & Setup

### Prerequisites

- **Operating System:** Windows 10/11, Ubuntu 20.04, or macOS
- **Python Runtime:** Python 3.10 to 3.12
- **Hardware Drivers for live capture:**
  - [Npcap](https://npcap.com/) on Windows
  - `libpcap` on Linux

### 1. Clone & Set Up Virtual Environment

```powershell
# Clone the repository
git clone https://github.com/your-username/AI_Intrusion_Detection.git
cd AI_Intrusion_Detection

# Initialize virtual environment
python -m venv venv

# Activate virtual environment (Windows PowerShell)
.\venv\Scripts\Activate.ps1

# Activate virtual environment (Linux / macOS)
# source venv/bin/activate
```

### 2. Install Required Dependencies

```powershell
pip install -r requirements.txt
```

---

# 🚀 Execution & Operational Modes

## Mode 1: Interactive Telemetry Dashboard (Recommended)

Launches the browser-based security operations dashboard featuring batch analysis, attack signature evaluation, and model metadata.

```powershell
streamlit run src/app.py
```

Access the interface in your browser at:

```text
http://localhost:8501
```

Navigate to **Batch Traffic Analysis** and upload:

```text
data/demo_sample_traffic.csv
```

This allows you to review:

- Automated classification metrics
- KPI counters
- Interactive Plotly distribution charts

---

## Mode 2: Live Network Packet Sniffing

Captures live frames passing through the physical network adapter, groups them into session flows, and performs real-time classification.

### Windows

The script must be executed in an **Administrator PowerShell** prompt.

```powershell
python src\live_detector.py
```

---

## Mode 3: Zero-Shot Ground-Truth Attack Harness

Validates that the serialized machine learning model correctly identifies every signature class without relying on external network traffic.

```powershell
python src\test_attack_detection.py
```

### Expected Verification Output

```text
================================================================
ACTUAL ATTACK TYPE   | AI PREDICTION      | STATUS
================================================================
Bots                 | Bots               | PASS (MATCH)
Brute Force          | Web Attacks        | MISMATCH
DDoS                 | DDoS               | PASS (MATCH)
DoS                  | DoS                | PASS (MATCH)
Normal Traffic       | Normal Traffic     | PASS (MATCH)
Port Scanning        | Port Scanning      | PASS (MATCH)
Web Attacks          | Web Attacks        | PASS (MATCH)
================================================================
```

---

## Mode 4: Synthetic Attack Generation (Stress Testing)

Generates high-frequency UDP socket traffic to stress-test the capture engine against volumetric denial-of-service signatures.

```powershell
python src\simulate_dos.py
```

---

# 🔬 Performance & Limitations Analysis

## Bidirectional Flow Extraction Trade-Off

### Offline Batch Analysis

Achieves **99.90% accuracy** because every record contains all **52 bidirectional statistical dimensions**, including:

- Inter-arrival times
- Backward packet intervals
- Window sizing

### Live Ingestion Bottleneck

Real-time Python packet parsing via Scapy captures single-direction packets immediately, but calculating complex backward flow statistics on the fly requires multi-threaded connection table management.

Features missing in lightweight live capture default toward benign baselines.

### Production Recommendation

High-throughput production deployments should decouple flow assembly from model inference by pairing:

- A dedicated **C++ flow collector**
- CICFlowMeter or IPFIX/NetFlow probes
- An asynchronous inference service such as:
  - FastAPI
  - Redis Queue

---

## 🧠 Machine Learning Pipeline

```text
Raw Network Traffic
        │
        ▼
Packet Capture / Flow Records
        │
        ▼
Feature Engineering
        │
        ▼
52 Flow Features
        │
        ▼
Normalization
        │
        ▼
RobustScaler
        │
        ▼
Random Forest Classifier
(50 Decision Trees)
        │
        ▼
Attack Classification
        │
        ├── Normal Traffic
        ├── DoS
        ├── DDoS
        ├── Port Scanning
        ├── Web Attacks
        ├── Brute Force
        └── Bots
```

---

## 🛡️ Supported Detection Classes

| Class | Description |
|---|---|
| **Normal Traffic** | Benign enterprise network traffic |
| **DoS** | Denial-of-service attacks |
| **DDoS** | Distributed/high-volume denial-of-service traffic |
| **Port Scanning** | Nmap scans, SYN scans and OS fingerprinting |
| **Web Attacks** | XSS, SQL injection and command injection |
| **Brute Force** | SSH/FTP credential stuffing |
| **Bots** | Botnet command-and-control communication |

---

## 📈 Key Capabilities

- Multi-class network intrusion detection
- Random Forest-based classification
- CICIDS2017 benchmark training
- 52 flow-level statistical features
- Batch PCAP/CSV traffic analysis
- Live packet sniffing with Scapy
- Session-based flow tracking
- Streamlit security telemetry dashboard
- Real-time terminal alerts
- Attack probability confidence
- Threat distribution visualization
- Ground-truth attack verification
- Synthetic UDP stress testing
- Serialized model and preprocessing artifacts

---

## ⚠️ Limitations

The current live packet ingestion implementation has an important trade-off:

- Offline flow records contain complete bidirectional statistics.
- Live Scapy capture receives packets incrementally.
- Some backward-flow statistics are therefore unavailable or incomplete during lightweight live capture.
- This can cause live classification to differ from offline benchmark performance.

For production-scale deployments, flow collection and ML inference should be separated using a dedicated high-performance flow collector and an asynchronous inference service.

---

## 📄 Project Components

| Component | Purpose |
|---|---|
| `app.py` | Streamlit telemetry and analysis dashboard |
| `live_detector.py` | Real-time packet capture and flow assembly |
| `test_attack_detection.py` | Attack classification verification |
| `simulate_dos.py` | Synthetic UDP stress testing |
| `random_forest_model.pkl` | Trained Random Forest model |
| `scaler.pkl` | Feature normalization/scaling |
| `label_encoder.pkl` | Attack class encoding |
| `feature_names.pkl` | 52-feature column mapping |
| `cicids2017_cleaned.csv` | Preprocessed CICIDS2017 dataset |
| `demo_sample_traffic.csv` | Dashboard demonstration dataset |

---

## 🔐 Security Considerations

This project is intended for controlled cybersecurity research, testing, and defensive network monitoring.

When using the live packet sniffer or synthetic traffic generator:

- Run testing only on networks and systems you are authorized to monitor.
- Use the synthetic attack generator only in controlled environments.
- Do not direct generated traffic toward third-party systems.
- Run packet capture with the required operating-system privileges.

---

## 📌 Summary

The **AI-Powered Network Intrusion Detection System (NIDS)** combines machine learning, network flow analysis, real-time packet capture, and interactive telemetry into a single intrusion detection workflow.

The system supports both **offline batch analysis** and **live packet ingestion**, using a Random Forest classifier trained on CICIDS2017 and a 52-feature flow representation.

Its architecture provides a foundation for experimenting with automated network threat classification while highlighting the performance and feature-completeness trade-offs between offline flow analysis and lightweight real-time packet capture.
