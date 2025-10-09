# Quic vs Dash on video streaming

**IUST project for comparison of the QUIC protocol with the DASH protocol**

## Table of Contents

- [Overview](#overview)  
- [Motivation](#motivation)  
- [Features / Goals](#features--goals)  
- [Architecture / Components](#architecture--components)   
- [Test Senarios](#senarios)  
  

---

## Overview

This project is a comparative study between **DASH** (Dynamic Adaptive Streaming over HTTP) and **QUIC** (Quick UDP Internet Connections) in the context of video streaming.  
The aim is to evaluate how DASH performs when the underlying transport is QUIC, with metrics like video quality, adaptive switching, data wastage, etc.

The repository contains scripts, experiments setup, video segments, and analysis code to run and compare these protocols.

---

## Motivation

- While DASH is widely used over HTTP/TCP, newer transport protocols like QUIC may offer advantages (e.g. lower latency, multiplexing, reduced head-of-line blocking) 
- The question is: *Does DASH over QUIC yield better streaming quality or adaptive performance than DASH over TCP?*  
- Understand the trade-offs (if any) — e.g. is there extra overhead, data wastage, or complexity in using QUIC?

---

## Features / Goals

- Automate experiment runs comparing DASH over TCP vs DASH over QUIC  
- Collect playback/streaming statistics (bandwidth usage, quality level changes, buffer, etc.)  
- Visualize / analyze metrics to draw conclusions  
- Provide a reproducible experimental setup  

---

## Architecture / Components

The project is organized as follows:

**Main Components:**
- **Topology Generator** - Generate the main topology for running the servers & client.
- **DASH Server** – Streams video segments over HTTP/TCP using standard DASH manifest files (`.mpd`).
- **QUIC Server** – Uses QUIC transport for low-latency video delivery with the same video segments.
- **Client Script** – Requests video segments adaptively and records playback performance metrics.
- **Network Emulator** – Uses Linux `tc/netem` to simulate delay, loss, or limited bandwidth.
- **Analysis Tools** – Extracts and visualizes key metrics (throughput, RTT, buffer level, etc.) for comparison.

---

## Senarios

**For test the topologoy in custom senario, we must config the `test_runner.py` file on client's CLI:**

`python3 test_runner.py -b <bandwidth(Mbps)> -d <delay(ms)> -j <jitter(ms)> -l <packet_loss(%)> [-t <cross_traffic_type(random, worst)>]
`
<br>
**The senario that I test:**
1.  python3 test_runner.py -b 20 -d 10
2.  python3 test_runner.py -b 10 -d 40 -j 10 -l 0.1
3.  python3 test_runner.py -b 5 -d 40 -j 10 -l 1 -t random
4.  python3 test_runner.py -b 5 -d 80 -j 30 -l 1
5.  python3 test_runner.py -b 2 -d 80 -j 30 -l 3 -t worst
6.  python3 test_runner.py -b 2 -d 80 -j 30 -l 3
7.  python3 test_runner.py -b 10 -d 80 -j 10 -l 0.1
8.  python3 test_runner.py -b 10 -d 40 -j 10 -l 1 -t random
9.  python3 test_runner.py -b 10 -d 40 -j 30 -l 1 -t random
10. python3 test_runner.py -b 10 -d 40 -j 10 -l 3
