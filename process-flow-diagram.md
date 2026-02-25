# SpectraPol-AI: Process Flow Diagram

This document provides a detailed visualization of the complete data flow through the SpectraPol-AI system, from physical cane scanning to operator action.

---

## Complete System Process Flow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         PHASE 1: PHYSICAL ACQUISITION                    │
└─────────────────────────────────────────────────────────────────────────┘

    [Sugarcane Delivery]
            │
            ▼
    ┌───────────────────┐
    │  Cane Shredder    │  ◄── Unloading & Chopping
    │  Heavy Duty       │      Exposes fibrous tissue
    └─────────┬─────────┘
              │
              ▼
    ┌─────────────────────────────────────────┐
    │   PRIMARY CANE CARRIER BELT             │
    │   (High-velocity conveyor transport)    │
    │                                         │
    │   ████████████████████████████►         │  ◄── Shredded cane flow
    └─────────────────┬───────────────────────┘
                      │
                      │ 0.5m mounting height
                      ▼
         ┌────────────────────────────┐
         │  SpectraPol-AI Sensor      │
         │  ┌──────────────────────┐  │
         │  │  Quartz Window       │  │
         │  │  (Optical Interface) │  │
         │  └──────────────────────┘  │
         │           ▲                │
         │           │                │
         │    ┌──────┴──────┐         │
         │    │ Air-Knife   │         │  ◄── Pneumatic cleaning
         │    │ (40-60 PSI) │         │      (Compressed air from mill)
         │    └─────────────┘         │
         └────────────┬───────────────┘
                      │
                      │ Broad-spectrum illumination
                      ▼
         ┌────────────────────────────┐
         │  18-Channel Reflectance    │
         │  Measurement @ 5Hz         │
         │                            │
         │  410nm ████ 940nm          │  ◄── Multi-spectral capture
         │  (18 discrete wavelengths) │
         └────────────┬───────────────┘
                      │
                      │ I2C Protocol
                      ▼

┌─────────────────────────────────────────────────────────────────────────┐
│                      PHASE 2: EDGE PROCESSING                            │
└─────────────────────────────────────────────────────────────────────────┘

         ┌────────────────────────────┐
         │   ESP32 Microcontroller    │
         │   ┌──────────────────────┐ │
         │   │ I2C Data Acquisition │ │
         │   │ @ 5Hz Sampling       │ │
         │   └──────────┬───────────┘ │
         │              │             │
         │   ┌──────────▼───────────┐ │
         │   │ Baseline Noise       │ │
         │   │ Reduction            │ │
         │   └──────────┬───────────┘ │
         └──────────────┼─────────────┘
                        │
                        │ Raw 18-channel array
                        ▼
         ┌────────────────────────────────────┐
         │  Python Edge AI Engine             │
         │  ┌──────────────────────────────┐  │
         │  │  Step 1: Preprocessing       │  │
         │  │  • SNV Transformation        │  │
         │  │  • Light scatter correction  │  │
         │  │  • Baseline normalization    │  │
         │  └──────────┬───────────────────┘  │
         │             │                      │
         │  ┌──────────▼───────────────────┐  │
         │  │  Step 2: Chemometric Model   │  │
         │  │  • PLS Regression            │  │
         │  │  • Latent variable isolation │  │
         │  │  • Sucrose concentration     │  │
         │  │  • Moisture detection        │  │
         │  └──────────┬───────────────────┘  │
         │             │                      │
         │  ┌──────────▼───────────────────┐  │
         │  │  Step 3: Output Generation   │  │
         │  │  • Pol % (e.g., 14.2%)       │  │
         │  │  • Moisture %                │  │
         │  │  • Quality flags             │  │
         │  │  • Confidence score          │  │
         │  └──────────┬───────────────────┘  │
         └─────────────┼─────────────────────┘
                       │
                       │ <200ms processing time
                       ▼
         ┌────────────────────────────┐
         │  JSON Payload Formation    │
         │  {                         │
         │    "pol": 14.2,            │
         │    "moisture": 68.5,       │
         │    "quality": "good",      │
         │    "timestamp": "...",     │
         │    "sensor_health": "ok"   │
         │  }                         │
         └────────────┬───────────────┘
                      │
                      │ MQTT / WebSocket
                      ▼

┌─────────────────────────────────────────────────────────────────────────┐
│                   PHASE 3: TELEMETRY & ACTION                            │
└─────────────────────────────────────────────────────────────────────────┘

         ┌────────────────────────────────────┐
         │   Cloud Backend Router             │
         │   (FastAPI / Node.js)              │
         │                                    │
         │   ┌──────────────────────────────┐ │
         │   │  Real-time Stream Handler    │ │
         │   │  • WebSocket connections     │ │
         │   │  • Data validation           │ │
         │   └──────────┬───────────────────┘ │
         └──────────────┼─────────────────────┘
                        │
                ┌───────┴────────┐
                │                │
                ▼                ▼
    ┌─────────────────┐   ┌──────────────────┐
    │  NoSQL Database │   │  Frontend Clients│
    │  (Firebase/     │   │  (WebSocket)     │
    │   MongoDB)      │   └────────┬─────────┘
    │                 │            │
    │  • Historical   │            ▼
    │    data logs    │   ┌──────────────────────────┐
    │  • Shift reports│   │  React Dashboard         │
    │  • ERP export   │   │  (Control Room UI)       │
    └─────────────────┘   │                          │
                          │  ┌────────────────────┐  │
                          │  │ Live Telemetry     │  │
                          │  │ ┌────────────────┐ │  │
                          │  │ │ Pol % Graph    │ │  │
                          │  │ │ ████████▲      │ │  │
                          │  │ │         │      │ │  │
                          │  │ │ 14.2%   │      │ │  │
                          │  │ └────────────────┘ │  │
                          │  └────────────────────┘  │
                          │                          │
                          │  ┌────────────────────┐  │
                          │  │ Threshold Alerts   │  │
                          │  │ 🔴 LOW POL WARNING │  │
                          │  │ Pol dropped to 12.1%│  │
                          │  └────────────────────┘  │
                          │                          │
                          │  ┌────────────────────┐  │
                          │  │ Sensor Health      │  │
                          │  │ ✓ Air-knife: OK    │  │
                          │  │ ✓ Optical: Clear   │  │
                          │  └────────────────────┘  │
                          └──────────┬───────────────┘
                                     │
                                     │ Visual alert
                                     ▼
                          ┌──────────────────────────┐
                          │  Mill Operator           │
                          │  (Control Room)          │
                          └──────────┬───────────────┘
                                     │
                                     │ Decision & Action
                                     ▼
                          ┌──────────────────────────┐
                          │  Process Optimization    │
                          │                          │
                          │  Actions:                │
                          │  • Adjust belt speed     │
                          │  • Increase imbibition   │
                          │    water flow            │
                          │  • Modify roller pressure│
                          │  • Divert low-quality    │
                          │    cane batches          │
                          └──────────────────────────┘
                                     │
                                     │ Feedback loop
                                     ▼
                          ┌──────────────────────────┐
                          │  Mill Process Adjustment │
                          │  (Real-time optimization)│
                          └──────────────────────────┘
```

---

## Detailed Phase Breakdown

### Phase 1: Physical Acquisition (Hardware Layer)

#### Step 1: Cane Shredding
- **Input**: Whole sugarcane stalks from delivery trucks
- **Process**: Heavy-duty shredder chops and exposes fibrous tissue
- **Output**: Shredded cane with exposed juice and fiber
- **Duration**: Continuous operation

#### Step 2: Conveyor Transport
- **Input**: Shredded cane from Step 1
- **Process**: High-velocity belt transport through mill
- **Speed**: Typically 2-5 m/s depending on mill capacity
- **Output**: Moving cane stream ready for scanning

#### Step 3: Optical Scanning
- **Sensor Position**: 0.5m above belt (optimal for reflectance)
- **Illumination**: Broad-spectrum LED array
- **Measurement**: 18-channel reflectance @ 5Hz
- **Key Wavelengths**:
  - 940nm → Moisture detection (O-H stretch overtone)
  - 810nm/860nm → Sucrose vs. bagasse (C-H stretch overtones)
  - 410-700nm → Additional spectral fingerprinting

#### Step 4: Self-Cleaning (Continuous)
- **Air-Knife System**: Pneumatic flat-sheet air curtain
- **Pressure**: 40-60 PSI from mill compressed air lines
- **Function**: Prevents sugar dust and moisture accumulation
- **Result**: Zero optical drift, consistent measurements

---

### Phase 2: Edge Processing (AI Layer)

#### Step 5: Data Ingestion
- **Hardware**: ESP32 microcontroller
- **Protocol**: I2C communication with AS7265x sensor
- **Sampling**: 5Hz continuous acquisition
- **Preprocessing**: Baseline noise reduction, outlier filtering
- **Output**: Clean 18-channel spectral array

#### Step 6: Chemometric AI
- **Algorithm**: Partial Least Squares (PLS) Regression
- **Preprocessing**: Standard Normal Variate (SNV) transformation
  - Corrects for light scattering variations
  - Normalizes spectral baseline
- **Model Execution**:
  - Extracts latent variables from 18 channels
  - Isolates sucrose-specific spectral signatures
  - Filters out bagasse interference
- **Outputs**:
  - Pol percentage (±0.2% accuracy)
  - Moisture percentage
  - Quality classification flags
  - Confidence score
- **Latency**: <200ms from raw data to result

---

### Phase 3: Telemetry & Action (Cloud & UI Layer)

#### Step 7: Cloud Transmission
- **Protocol**: MQTT or WebSocket
- **Payload**: JSON format with Pol %, moisture %, health metrics
- **Frequency**: Real-time stream (5Hz)
- **Reliability**: Automatic reconnection, local buffering on disconnect

#### Step 8: Real-Time Visualization
- **Frontend**: React-based control room dashboard
- **Charts**: Live telemetry with Recharts/Chart.js
- **Features**:
  - Rolling time-series graphs (last 5 minutes, 1 hour, shift)
  - Color-coded quality zones (green/yellow/red)
  - Sensor health indicators
  - Air-knife status monitoring
- **Update Rate**: Sub-second UI refresh

#### Step 9: Operator Action
- **Alert System**:
  - Automated threshold monitoring
  - Visual alerts (flashing red) for low Pol
  - Audio alerts (optional) for critical drops
- **Operator Response**:
  - Review current Pol trend
  - Assess mill operating parameters
  - Adjust process variables:
    - Belt speed (slower = better extraction)
    - Imbibition water (more water = better juice extraction)
    - Roller pressure (higher = more crushing)
  - Divert low-quality batches if necessary
- **Feedback Loop**: Immediate impact visible on dashboard (30-60 seconds)

---

## Data Flow Summary

| Stage | Input | Processing | Output | Latency |
|-------|-------|------------|--------|---------|
| **Scanning** | Shredded cane | 18-channel reflectance @ 5Hz | Raw spectral array | 200ms |
| **Edge AI** | Spectral array | SNV + PLS regression | Pol %, moisture % | <200ms |
| **Transmission** | JSON payload | MQTT/WebSocket | Cloud data stream | 50-100ms |
| **Visualization** | Data stream | React rendering | Live dashboard | <100ms |
| **Total** | Physical cane | End-to-end pipeline | Operator insight | **<500ms** |

---

## Error Handling & Fault Tolerance

### Sensor Failures
```
Sensor Error Detected
        │
        ▼
┌───────────────────┐
│ ESP32 Diagnostics │
└────────┬──────────┘
         │
    ┌────┴────┐
    │         │
    ▼         ▼
[I2C Fail] [Optical Fail]
    │         │
    ▼         ▼
Alert     Air-Knife
Dashboard  Check
```

### Network Disconnection
```
Cloud Connection Lost
        │
        ▼
┌───────────────────┐
│ Local Buffering   │
│ (ESP32 memory)    │
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│ Reconnection      │
│ Attempts (3x)     │
└────────┬──────────┘
         │
    ┌────┴────┐
    │         │
    ▼         ▼
[Success] [Failure]
    │         │
    ▼         ▼
Resume    Operator
Stream    Alert
```

### Calibration Drift
```
Monthly Validation Check
        │
        ▼
┌───────────────────┐
│ Reference Sample  │
│ Measurement       │
└────────┬──────────┘
         │
    ┌────┴────┐
    │         │
    ▼         ▼
[Within   [Drift
 Spec]    Detected]
    │         │
    ▼         ▼
Continue  Recalibrate
Normal    Required
```

---

## Performance Optimization Points

1. **Edge Processing**: Eliminates cloud latency for critical measurements
2. **Air-Knife**: Prevents optical degradation, maintains accuracy
3. **PLS Algorithm**: Handles collinearity, robust to noise
4. **WebSocket**: Low-latency real-time data streaming
5. **Local Buffering**: Ensures no data loss during network issues

---

**Document Version**: 1.0.0  
**Last Updated**: February 2026  
**Related Documents**: [README.md](./README.md) | [architecture-diagram.md](./architecture-diagram.md) | [system-design.md](./system-design.md)
