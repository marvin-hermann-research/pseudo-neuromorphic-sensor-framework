# Pseudo-Neuromorphic Sensor Framework

## Overview
This repository implements a **universal pseudo-neuromorphic sensor framework** designed for a wide range of microcontrollers, including Arduino Nano, ESP32, STM32, and potentially smaller MCUs.  
The core idea is to **convert continuous sensor readings into event-based spikes** rather than transmitting dense raw data streams. This enables efficient, low-latency, and energy-aware sensing suitable for robotics, embedded systems, and research applications.

---

## Goals & Scope

- **Goal:** Develop a hardware-agnostic framework for event-driven pseudo-neuromorphic sensing.
- **Core Principle:** Transform sensor data into discrete **events** (spike-like impulses) triggered by meaningful changes.
- **Minimum Viable Product (MVP):**
  - 1x single-value sensor (e.g., light, NTC)
  - 1x multi-channel sensor (e.g., IMU, 3-axis)
- **Metrics for Evaluation:**
  - Event latency
  - Events per second
  - CPU utilization
  - RAM footprint
  - Energy consumption

---

## Architectural Principles

### 1. Core ↔ Hardware Separation
- **Core:** Fully hardware-independent, contains:
  - Event definition
  - Ringbuffer / Scheduler API
  - Spike detectors (Δ-threshold, EMA, CUSUM)
- **Profiles / HAL:** MCU-specific implementations:
  - Timer, ADC, DMA, sampling rates
  - Buffer allocation and memory management

### 2. Sensor Layer
- All sensors implement `ISensor<T>` interface.
- MCU-specific ADC/I²C drivers reside only in Profile/HAL layer.

### 3. Transport Layer
- Interface: `Transport.write(Event*, size_t)`
- Implementations:
  - UART (binary/raw)
  - micro-ROS
  - CAN, BLE
- Core interacts only via interface → remains platform-independent.

### 4. Event Structure
```cpp
struct Event {
  uint32_t t_us;      // Timestamp (wrap-aware)
  uint8_t  sensor_id;
  uint8_t  channel;
  int8_t   polarity;  // -1, 0, +1
  int16_t  magnitude; // quantized, fixed-point
};
```

### 5. Sampling Backbone

- Timer / ISR writes raw sensor samples to a ringbuffer.
- Main loop executes:
  - Event detection
  - Spike encoding
  - Transport dispatch
- ISR is extremely lightweight; CPU-intensive computation occurs in main loop only.

### 6. Spike Detectors

- Implemented in fixed-point (for low-resource MCUs) or float (for high-performance MCUs)
- Interchangeable strategies using templates or factories
- Adaptive thresholds configurable at runtime via UART

### 7. Dynamic Configuration

- Compile-time profiles (Nano, ESP32, STM32, …)
- Runtime parameter updates via UART, e.g.:
  `SET th=25,alpha=0.03,ref_us=300`
- Sampling rate auto-adjusts to MCU clock

---

## Repository Structure

```bash
/src
  core/
    event.hpp
    ring_buffer.hpp
    spike_detectors.hpp
    encoder.hpp
    scheduler.hpp       // API, implementation per profile
  sensors/
    isensor.hpp
    light_adc.hpp       // MVP
    imu_dummy.hpp       // MVP
  transports/
    transport.hpp
    uart_raw.hpp
    ros2_msgs.hpp
  profiles/
    nano_profile.hpp
    esp32_profile.hpp
    stm32_profile.hpp
    hal/                // Timer, ADC, DMA abstraction
/examples
  light_mono_event/
  imu_tri_event/
/bench
  scripts/              // CSV parser, host benchmarking
```

**Key Principles:**
- Core is fully MCU-independent
- All MCU-specific logic exists only in Profiles/HAL
- Interfaces abstract sensors, transports, and scheduling

---

## Features

- Modular, hardware-agnostic architecture
- Event-based encoding reduces bandwidth while emphasizing salient changes
- Configurable spike detection algorithms
- Unified event format compatible with ROS2 and other middleware
- Hardware profiles for low- and high-resource MCUs
- Runtime-adjustable parameters and sampling rates

## Future Directions

- Extended support for STM32, including DMA-based sampling
- Full micro-ROS integration with standardized `EventArray` messages
- Sensor-specific optimizations (e.g., per-axis IMU analysis, dynamic window sizing)
- Benchmarks comparing event-driven vs. continuous sensing in real robotic tasks

## Disclaimer

This framework is **experimental** and intended for research and prototyping.  
It is **not industrial-grade** but serves as a bridge between neuromorphic sensing principles and low-cost, accessible hardware, providing a flexible platform for algorithm testing and robotics integration.
