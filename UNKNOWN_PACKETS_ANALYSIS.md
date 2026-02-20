# Unknown Packet Analysis Report

## Executive Summary

After analyzing the pcap file and cross-referencing with documentation, I found that **0xc8 and 0xcf are truly unknown packets** that are not documented in either:
- The pycozmo protocol declaration
- The Anki cozmoclad SDK
- Any online documentation

## What Are These Packets?

### 0xc8 Packet
- **Size (from robot)**: Variable! 3-254 bytes (NOT the expected 29 bytes)
- **Size (from engine)**: Variable! 163-199 bytes
- **Count**: 122 from robot, 350 from engine
- **Direction**: Bidirectional (both robot and engine send this)
- **Hypothesis**: This is likely **encrypted or compressed data** since:
  - All 29 bytes are highly variable (no constant fields)
  - Sizes vary widely
  - Sent frequently during entire session (~6.5 packets/sec average)

### 0xcf Packet  
- **Size (from robot)**: Variable! 2-253 bytes (NOT the expected 8 bytes)
- **Size (from engine)**: Variable! 178-254 bytes
- **Count**: 112 from robot, 213 from engine
- **Direction**: Bidirectional
- **Hypothesis**: Similar to 0xc8, this appears to be **variable-length data** sent throughout the session

## Key Findings

### 1. Protocol Documentation May Be Outdated
The pycozmo protocol documentation lists:
```
0xc8: min=29, max=29
0xcf: min=8, max=8
```

But the actual pcap shows variable sizes, suggesting:
- These packets may have been changed in firmware updates after the protocol was documented
- The documentation captured a specific firmware version's behavior
- These might be different packet types that reuse the same IDs

### 2. Timing Analysis
- **0xc8**: Appears ~6.5 times/second throughout the 75-second session
- **0xcf**: Appears ~4.1 times/second throughout the 81-second session
- Both appear immediately after connection (at 1.02s and 1.09s)
- Neither correlates specifically with motor calibration events

### 3. Data Pattern Analysis
For 0xc8 (29-byte packets):
- Every byte position has 90+ unique values
- No constant bytes (unlike structured packets)
- No obvious integer or float patterns
- Data appears random/encrypted

### 4. Relationship to Known Packets
- **NOT the same as cozmoclad's 0xc8**: The cozmoclad library defines 0xc8 as "RegisterOnboardingComplete" but that's for the **app-to-engine protocol**, not the robot protocol
- **NOT motor calibration**: 0xd1 is MotorCalibration (well-documented)
- **NOT NV storage**: 0xcd is NvStorageOpResult
- **NOT IMU data**: RobotState (0xf0) already contains accel/gyro

## Hypotheses About Purpose

### Hypothesis 1: IMU Calibration Data (0xc8)
The NVEntry_IMUInfo tag (0x80000006) exists in NV storage, suggesting IMU calibration is stored. 0xc8 could be:
- Real-time IMU calibration updates
- Gyro/accel bias compensation data
- Temperature compensation tables

### Hypothesis 2: Debug/Logging Data (0xcf)
Similar to DebugData (0xb0) which carries logging information, 0xcf could be:
- Motor encoder readings
- Power management data
- Debug telemetry

### Hypothesis 3: Encryption/Authentication
The variable sizes and random data could indicate:
- Encrypted control channel data
- Authentication handshakes
- Firmware integrity checks

### Hypothesis 4: Variable-Length Protocol Extension
These might be packets that use a different framing scheme:
- First byte = sub-type
- Remaining bytes = payload
- This would explain the variable sizes

