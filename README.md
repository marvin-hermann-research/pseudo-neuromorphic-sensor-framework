# Pseudo-Neuromorphic Sensor Framework

## Overview
This repository provides a lightweight framework for building **pseudo-neuromorphic sensors** on low-cost microcontrollers such as the Arduino Nano and ESP32.  
The central idea is to transform continuous sensor readings into **event-based signals** by applying temporal windowing, spike detection, and adaptive encoding.  

Instead of transmitting raw, high-frequency data streams, the framework outputs **sparse, spike-like impulses** only when meaningful changes occur.  
This mimics principles of biological neuromorphic sensing while remaining accessible for low-cost hardware and adaptable to different sensor types.

---

## Motivation
Conventional sensors typically deliver continuous, dense data streams, which consume bandwidth and processing capacity while often carrying redundant information.  
In contrast, neuromorphic sensing emphasizes **event-driven processing**, where signals are generated only when significant changes occur.  

This framework translates these principles into the domain of hobbyist and research microcontrollers by combining:

- High-frequency polling of standard sensors  
- Temporal windowing for local analysis  
- Spike-based detection of significant changes  
- Adaptive encoding to reduce redundancy while preserving salient events  

The result is a sensor interface that is more efficient, event-driven, and closer to true neuromorphic devices, making it highly suitable for **robotics, embedded systems, and ROS2-based platforms**.

---

## Architectural Principles
The framework is structured around several core layers:

### Sensors (Input Layer)
- Abstract interfaces for common sensor types (single-value sensors such as light or temperature, and multi-channel sensors such as IMUs).  
- Each sensor is polled at a high frequency, independent of the output event rate.

### Spike Detectors (Processing Layer)
Multiple interchangeable strategies for event extraction, including:
- Simple threshold + hysteresis  
- Exponential moving average (EMA) with adaptive thresholds  
- Lightweight drift/shift detection methods (e.g., CUSUM)  

These algorithms are optimized for constrained devices through fixed-point arithmetic and minimal memory usage.

### Event Representation (Encoding Layer)
Sensor changes are encoded into a normalized event format consisting of:
- Timestamp (µs resolution)  
- Sensor ID and channel  
- Polarity (+/– change)  
- Magnitude (quantized intensity)  

This unified event structure enables compatibility with ROS2 and other event-driven systems.

### Transport (Output Layer)
Events are serialized and transmitted efficiently:
- On constrained devices (e.g., Arduino Nano), a binary UART protocol with framing and checksums is used.  
- On more capable devices (e.g., ESP32), integration with **micro-ROS** enables direct publishing of event arrays to ROS2 topics.  

### Profiles (Hardware Adaptation)
Device-specific profiles define buffer sizes, maximum sampling rates, and memory allocations, ensuring stable operation on both low-resource (Nano) and high-performance (ESP32) targets.

---

## Features
- Modular architecture adaptable to different sensor modalities (light, distance, IMU, pressure, etc.)  
- Event-based encoding that reduces bandwidth and highlights salient changes  
- Configurable spike-detection algorithms optimized for real-time operation  
- Unified event format for integration with ROS2 and robotics middleware  
- Hardware profiles for both low-resource MCUs and higher-performance platforms  

---

## Future Directions
- Extended support for ESP32 with DMA-based sampling and FreeRTOS task separation  
- Full **micro-ROS** integration with standard ROS2 message types (`EventArray`)  
- Sensor-specific optimization strategies (e.g., dynamic window sizes, per-axis IMU analysis)  
- Benchmarks comparing event-driven vs. continuous sampling in robotic control tasks  

---

## References / Inspiration
- **Steffen, L., Reichard, D., Weinland, J., Kaiser, J., Roennau, A., & Dillmann, R. (2019).**  
  *Neuromorphic Stereo Vision: A Survey of Bio-Inspired Sensors and Algorithms.*  
  Frontiers in Neurorobotics, 13.  
  DOI: [10.3389/fnbot.2019.00028](https://doi.org/10.3389/fnbot.2019.00028)  

- **MaiRo Lab / IMI, KIT**  
  Research on neuromorphic sensing and robotic integration, which inspired this framework’s attempt to translate neuromorphic principles into **low-cost, accessible hardware platforms**.  
  More information: [https://www.imi.kit.edu/4418.php](https://www.imi.kit.edu/4418.php)

---

## Disclaimer
This framework is **experimental** and designed as a research exploration of event-based sensing on commodity hardware.  
It is not optimized for industrial deployment, but aims to **bridge the gap between neuromorphic sensing principles and low-cost prototyping**, providing both a testing ground for algorithms and a pathway toward robotics integration.
