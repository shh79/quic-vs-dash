# Quic vs Dash on Video Streaming

**IUST project for comparison of the QUIC protocol with the DASH protocol**

## Table of Contents

- [Overview](#overview)  
- [Directory Structure](#directory-structure)  
- [Setup Instructions](#setup-instructions)  
- [Run Mininet Topology](#run-mininet-topology)   
- [Analyze the Results](#analyze-the-results) 
- [Conclusion](#conclusion) 

## Overview

This project compares the QUIC (Quick UDP Internet Connections) and DASH (Dynamic Adaptive Streaming over HTTP) protocols in the context of video streaming. The aim is to evaluate their performance, reliability, and efficiency when streaming a sample video under various network conditions.

The repository contains scripts, experiments setup, video segments, and analysis code to run and compare these protocols.

## Directory Structure
```
├── streaming
│   ├── dash_server.py
│   ├── quic_server.py
│   ├── test_runner.py
│   ├── random_traffic.py
│   ├── worst_traffic.py
│   ├── topo.py
│   ├── results/*.csv, *.mp4, *.txt
│   ├── qlog/*.qlog
│   ├── plots/
│   ├── analise_results.py
│   ├── dash_content/
│   └── senario-result-repo/
│       ├── CDF_startup_delay/
│       │   ├── dash/*.csv
│       │   ├── quic/*.qlog
│       └── ├── compare_startup_delay.py
├── video_segments/*.mp4
├── cert.pem
├── key.pem
```

* **`streaming/`**: Contains all scripts related to server setup, traffic generation, and running the video clients.

  * **`topo.py`**: Generates the Mininet topology for the project.
  * **`quic_server.py`**: Runs the QUIC server on `s1`.
  * **`dash_server.py`**: Runs the DASH server on `s2`.
  * **`test_runner.py`**: Configures the network and runs the clients for both protocols (`quic-client.py` and `dash-client.py`): Runs the test script client on `c1`
  * **`random_traffic.py`**: Generates random traffic (1–5 flows) using TCP or UDP that called from `test_runner.py` if it is needed.
  * **`worst_traffic.py`**: Simulates other video-streaming traffic using variant UDP flows that called from `test_runner.py` if it is needed.
  * **`analise_results.py`**: Analyzes the results by reading logs from `results/` and `qlog/` folders, then plots the data and saves it as image files in the `plots/` folder.
  * **`results/`**: Stores logs of both protocols and the received video files.
  * **`qlog/`**: Stores log files specific to the QUIC protocol.
  * **`plots/`**: Stores the charts generated from the analysis in PNG format.
  * **`dash_content/`**: Contains the segments of the sample video in 8-second chunks at three quality levels: 360p, 720p, and 1080p. It also includes the `manifest.mpd` file, which is required for the DASH protocol to provide the necessary information about video segments suitable for different network conditions. this folder used from `dash` server that runned on `s2`.
  * **`senario-result-repo/`**: Contains test results for different scenarios.

    * **`CDF_startup_delay/`**: Holds the results of 10 different test scenarios for both QUIC and DASH protocols, each with `.csv` and `.qlog` files.
    * **`CDF_startup_delaycompare_startup_delay.py`**: Script to generate `startup_delay_quic_dash.csv` from the scenario results and plot a chart `CDF_startup_delay.png`.

* **`video_segments/`**: Contains sample video segments in different qualities (360p, 720p, 1080p), each with an 8-second duration. this folder used from `quic` server that runned on `s1`.

* **`cert.pem`** & **`key.pem`**: SSL certificates for securing the QUIC protocol.

## Setup Instructions

### Prerequisites

Ensure you have the following installed:

* Mininet
* Python 3.7 or higher
* OpenSSL (for generating certificates)
* Iperf3 (for traffic generation)
* pandas (for data diagram)
* matplotlib (for plot charts)
* seaborn (for plot charts)
* numpy (for calculation logs)

### Installation

Clone the repository:

```bash
git clone https://github.com/shh79/quic-vs-dash.git
cd quic-vs-dash
```

Install required Python packages:

```bash
pip install -r requirements.txt
```

For Generate SSL certificates for QUIC runed:

```bash
openssl req -new -x509 -days 365 -nodes -out cert.pem -keyout key.pem
```

## Run Mininet Topology

First, you need to generate the Mininet topology by running:

```bash
sudo python3 streaming/topo.py
```

### Running the Servers

* On `s1` (QUIC server):

```bash
python3 streaming/quic_server.py
```

* On `s2` (DASH server):

```bash
python3 streaming/dash_server.py
```

* On `s3` (Background server running `iperf3`):

```bash
iperf3 -s
```

### Running the Test

Run the test runner on `c1` (client):

```bash
python3 streaming/test_runner.py
```
The `test_runner.py` script will configure the link between `c1` and the router, set up background traffic using `iperf3` rules, and then run both the QUIC and DASH clients (`quic-client.py` and `dash-client.py`) to request the video streams from their respective servers.

---

### Senarios

**For test the topologoy in custom senario, we must config the `test_runner.py` file on client's CLI:**

```bash
python3 test_runner.py -b <bandwidth(Mbps)> -d <delay(ms)> -j <jitter(ms)> -l <packet_loss(%)> -t <cross_traffic_type(random, worst)>
```

### The senario that I test:
* python3 test_runner.py -b 20 -d 10
* python3 test_runner.py -b 10 -d 40 -j 10 -l 0.1
* python3 test_runner.py -b 5 -d 40 -j 10 -l 1 -t random
* python3 test_runner.py -b 5 -d 80 -j 30 -l 1
* python3 test_runner.py -b 2 -d 80 -j 30 -l 3 -t worst
* python3 test_runner.py -b 2 -d 80 -j 30 -l 3
* python3 test_runner.py -b 10 -d 80 -j 10 -l 0.1
* python3 test_runner.py -b 10 -d 40 -j 10 -l 1 -t random
* python3 test_runner.py -b 10 -d 40 -j 30 -l 1 -t random
* python3 test_runner.py -b 10 -d 40 -j 10 -l 3

---

## Analyze the Results

After running the test and receiving video from both protocols, you can analyze the results by running:

```bash
python3 streaming/analise_results.py
```

This script will:

* Read the logs from `streaming/results/` and `streaming/qlog/` folders.
* Plot the following performance metrics:

  * **Bitrate vs Time**
  * **Buffer Level vs Time**
  * **qLog RTT vs Time**
  * **Stall Timeline (Gantt-like)**
  * **Throughput vs Time**
* Save the charts as PNG images in the `streaming/plots/` folder.

### Scenario Testing and Analysis

The **`CDF_startup_delay`** folder contains the results for 10 different scenarios for both QUIC and DASH protocols, including `.csv` and `.qlog` files. To analyze these results and compare the startup delay between the two protocols, run the following script:

```bash
python3 streaming/senario-result-repo/CDF_startup_delay/compare_startup_delay.py
```

This will:

* Generate a `startup_delay_quic_dash.csv` file with the results of the startup delay comparison.
* Plot the **CDF of Startup Delay** and save it as an image (`CDF_startup_delay.png`) in the `streaming/senario-result-repo/CDF_startup_delay` folder.

### Logs and Results

* Logs for both protocols will be stored in the `streaming/results/` folder.
* The received video files will be stored in the `streaming/results/` folder.
* QUIC-specific log files (qlogs) will be stored in the `streaming/qlog/` folder.
* Charts and plots generated from the analysis will be saved in the `streaming/plots/` folder.
* Scenario results, including `.csv` and `.qlog` files, will be stored in the `streaming/senario-result-repo/CDF_startup_delay/` folder in seprated folders.

## Conclusion

This project offers a comprehensive comparison between the QUIC and DASH protocols for video streaming. By simulating various network conditions and traffic patterns, it provides valuable insights into the performance and reliability of these protocols.

#### For more detail of project read the thesis file that named `Video Streaming Over Quic Protocol - Hessam Hosseini - October 2025.pdf` that located in `Thesis` folder
---
