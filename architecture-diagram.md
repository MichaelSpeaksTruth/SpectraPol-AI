# SpectraPol-AI: System Architecture

This document provides a comprehensive view of the SpectraPol-AI system architecture, detailing all layers, components, interfaces, and data flows.

---

## High-Level Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        SPECTRAPOL-AI ARCHITECTURE                        │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│                    LAYER 1: PHYSICAL MILL ENVIRONMENT                    │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
              ┌─────────────────────────────────────┐
              │   PRIMARY CANE CARRIER BELT         │
              │   (Shredded cane @ 2-5 m/s)         │
              └──────────────┬──────────────────────┘
                             │ Reflected Light
                             ▼

┌─────────────────────────────────────────────────────────────────────────┐
│              LAYER 2: EDGE HARDWARE & SENSING LAYER                      │
└─────────────────────────────────────────────────────────────────────────┘

              ┌─────────────────────────────────────┐
              │  IP67 INDUSTRIAL ENCLOSURE          │
              │  ┌───────────────────────────────┐  │
              │  │ Pneumatic Air-Knife System    │  │
              │  │ (40-60 PSI compressed air)    │  │
              │  └───────────────────────────────┘  │
              │  ┌───────────────────────────────┐  │
              │  │ Quartz Optical Window         │  │
              │  │ (Scratch-resistant)           │  │
              │  └───────────────────────────────┘  │
              │  ┌───────────────────────────────┐  │
              │  │ AS7265x Triad Sensor          │  │
              │  │ • 18 Wavelengths (410-940nm)  │  │
              │  │ • 5Hz Sampling Rate           │  │
              │  │ • I2C Interface               │  │
              │  └──────────┬────────────────────┘  │
              │             │                       │
              │  ┌──────────▼────────────────────┐  │
              │  │ ESP32 Microcontroller         │  │
              │  │ • Dual-core 240MHz            │  │
              │  │ • WiFi + Bluetooth            │  │
              │  │ • I2C Master                  │  │
              │  │ • MQTT Client                 │  │
              │  └──────────┬────────────────────┘  │
              └─────────────┼───────────────────────┘
                            │ Raw 18-channel array
                            │ via MQTT
                            ▼

┌─────────────────────────────────────────────────────────────────────────┐
│              LAYER 3: AI & DATA PROCESSING LAYER                         │
└─────────────────────────────────────────────────────────────────────────┘

              ┌─────────────────────────────────────┐
              │  PYTHON EDGE / BACKEND SERVER       │
              │                                     │
              │  ┌───────────────────────────────┐  │
              │  │ Data Preprocessing Module     │  │
              │  │ • SNV Transformation          │  │
              │  │ • Baseline Correction         │  │
              │  │ • Outlier Filtering           │  │
              │  └──────────┬────────────────────┘  │
              │             │                       │
              │  ┌──────────▼────────────────────┐  │
              │  │ Chemometric Engine            │  │
              │  │ • Scikit-learn PLS Regression │  │
              │  │ • Latent Variable Extraction  │  │
              │  │ • Sucrose Isolation           │  │
              │  └──────────┬────────────────────┘  │
              │             │                       │
              │  ┌──────────▼────────────────────┐  │
              │  │ Output Generator              │  │
              │  │ • Pol % Calculation           │  │
              │  │ • Moisture % Calculation      │  │
              │  │ • Quality Flags               │  │
              │  │ • Confidence Scoring          │  │
              │  └──────────┬────────────────────┘  │
              └─────────────┼───────────────────────┘
                            │ JSON Payload
                            │ via WebSocket
                            ▼

┌─────────────────────────────────────────────────────────────────────────┐
│                 LAYER 4: CLOUD & STORAGE LAYER                           │
└─────────────────────────────────────────────────────────────────────────┘

    ┌─────────────────────────┐         ┌─────────────────────────┐
    │  REALTIME ROUTER        │◄───────►│  NOSQL DATABASE         │
    │  (FastAPI / Node.js)    │         │  (Firebase / MongoDB)   │
    │                         │         │                         │
    │  • WebSocket Server     │         │  • Historical Data      │
    │  • MQTT Broker          │         │  • Shift Reports        │
    │  • REST API Endpoints   │         │  • Calibration Logs     │
    │  • Data Validation      │         │  • Alert History        │
    │  • Load Balancing       │         │  • User Management      │
    └────────────┬────────────┘         └─────────────────────────┘
                 │                                    │
                 │ WebSocket Stream                   │ Query/Store
                 ▼                                    ▼
              [Connected Clients]              [ERP Integration]

┌─────────────────────────────────────────────────────────────────────────┐
│              LAYER 5: CLIENT & PRESENTATION LAYER                        │
└─────────────────────────────────────────────────────────────────────────┘

              ┌─────────────────────────────────────┐
              │  REACT CONTROL ROOM DASHBOARD       │
              │                                     │
              │  ┌───────────────────────────────┐  │
              │  │ Real-Time Telemetry Display   │  │
              │  │ • Live Line Charts (Recharts) │  │
              │  │ • Pol % Trend Graph           │  │
              │  │ • Moisture % Graph            │  │
              │  │ • Quality Score Indicator     │  │
              │  └───────────────────────────────┘  │
              │  ┌───────────────────────────────┐  │
              │  │ Alert & Notification System   │  │
              │  │ • Threshold Monitoring        │  │
              │  │ • Visual Alerts (Red/Green)   │  │
              │  │ • Audio Alerts (Optional)     │  │
              │  │ • Alert History Log           │  │
              │  └───────────────────────────────┘  │
              │  ┌───────────────────────────────┐  │
              │  │ Sensor Health Monitor         │  │
              │  │ • Air-Knife Status            │  │
              │  │ • Optical Window Clarity      │  │
              │  │ • Network Connectivity        │  │
              │  │ • Calibration Status          │  │
              │  └───────────────────────────────┘  │
              │  ┌───────────────────────────────┐  │
              │  │ Historical Analytics          │  │
              │  │ • Shift Reports               │  │
              │  │ • Batch Comparisons           │  │
              │  │ • Trend Analysis              │  │
              │  │ • Export to CSV/PDF           │  │
              │  └───────────────────────────────┘  │
              └─────────────────────────────────────┘
```

---

## Layer-by-Layer Component Details

### Layer 1: Physical Mill Environment

**Purpose**: The industrial setting where sugarcane processing occurs.

**Components**:
- Cane shredder (existing mill equipment)
- Primary cane carrier belt (existing mill equipment)
- Mounting infrastructure for sensor enclosure

**Environmental Conditions**:
- Temperature: 0°C to 50°C
- Humidity: 0-95% RH (non-condensing)
- Dust: High concentration of sugar dust and bagasse particles
- Vibration: Moderate from mill machinery
- Lighting: Variable ambient light

---

### Layer 2: Edge Hardware & Sensing Layer

**Purpose**: Capture multi-spectral reflectance data from moving cane in harsh industrial conditions.

#### 2.1 Industrial Enclosure

**Specifications**:
- Material: Die-cast aluminum
- Rating: IP67 (dust-tight, water-resistant)
- Dimensions: ~150mm x 100mm x 80mm
- Mounting: Adjustable bracket for 0.5m height above belt
- Cable glands: IP67-rated for power and pneumatic connections

**Features**:
- Thermal management (passive cooling fins)
- Vibration dampening mounts
- EMI shielding for industrial environment

#### 2.2 Pneumatic Air-Knife System

**Purpose**: Self-cleaning mechanism to prevent optical window contamination.

**Specifications**:
- Air source: Mill compressed air lines (40-60 PSI)
- Nozzle type: Flat-sheet air curtain
- Flow rate: Adjustable via inline regulator
- Coverage: Full quartz window surface
- Operation: Continuous during scanning

**Benefits**:
- Zero optical drift
- No manual cleaning required
- Extends calibration intervals
- Maintains measurement accuracy

#### 2.3 Quartz Optical Window

**Specifications**:
- Material: Fused quartz (high purity)
- Thickness: 3mm
- Coating: Anti-reflective (AR) for 400-1000nm
- Scratch resistance: 7 Mohs hardness
- Transmission: >90% across sensor wavelengths

#### 2.4 AS7265x Triad Multi-Spectral Sensor

**Specifications**:
- Manufacturer: AMS (now ams OSRAM)
- Configuration: 3-sensor array (AS72651 + AS72652 + AS72653)
- Total channels: 18 discrete wavelengths
- Wavelength range: 410nm - 940nm
- Spectral resolution: ~20nm FWHM per channel
- Integration time: Adjustable (50ms - 1000ms)
- Interface: I2C (address 0x49)
- Power: 3.3V, <100mA

**Wavelength Mapping**:
```
AS72651 (Visible):
  410nm, 435nm, 460nm, 485nm, 510nm, 535nm

AS72652 (Visible-NIR):
  560nm, 585nm, 610nm, 645nm, 680nm, 705nm

AS72653 (NIR):
  730nm, 760nm, 810nm, 860nm, 900nm, 940nm
```

**Key Wavelengths for Chemometrics**:
- 940nm: O-H stretch overtone (moisture detection)
- 810nm/860nm: C-H stretch overtones (sucrose vs. bagasse)
- 410-700nm: Additional spectral fingerprinting

#### 2.5 ESP32 Microcontroller

**Specifications**:
- SoC: ESP32-WROOM-32 or ESP32-WROVER
- CPU: Dual-core Xtensa LX6 @ 240MHz
- RAM: 520KB SRAM
- Flash: 4MB (minimum)
- Connectivity: WiFi 802.11 b/g/n, Bluetooth 4.2
- I2C: Hardware I2C master (100kHz - 400kHz)
- GPIO: Multiple pins for status LEDs, buttons
- Power: 3.3V logic, 5V input via regulator

**Firmware Responsibilities**:
- I2C communication with AS7265x sensor
- 5Hz data acquisition loop
- Baseline noise reduction
- MQTT client for data transmission
- WiFi connection management
- OTA (Over-The-Air) firmware updates
- Watchdog timer for fault recovery

**Communication Protocol**:
- MQTT broker connection
- Topic structure: `spectrapol/{mill_id}/{sensor_id}/data`
- QoS: Level 1 (at least once delivery)
- Payload: JSON format with timestamp

---

### Layer 3: AI & Data Processing Layer

**Purpose**: Transform raw spectral data into actionable chemical measurements using edge AI.

#### 3.1 Data Preprocessing Module

**Algorithms**:
- **Standard Normal Variate (SNV)**: Corrects for light scattering variations
  ```
  SNV(x) = (x - mean(x)) / std(x)
  ```
- **Baseline Correction**: Removes spectral baseline drift
- **Outlier Filtering**: Removes anomalous readings (e.g., belt gaps)

**Implementation**:
- Language: Python 3.8+
- Libraries: NumPy for array operations
- Execution: Real-time processing (<50ms)

#### 3.2 Chemometric Engine

**Algorithm**: Partial Least Squares (PLS) Regression

**Why PLS?**
- Handles high collinearity in multi-spectral data
- Extracts latent variables correlated with target (Pol %)
- Robust to noise and irrelevant spectral features
- Industry standard for chemometrics

**Model Training**:
- Training data: 100+ reference samples with known Pol values
- Cross-validation: Leave-one-out or k-fold
- Latent variables: Typically 3-5 components
- Validation: R² > 0.95, RMSEP < 0.2%

**Implementation**:
- Library: Scikit-learn (`sklearn.cross_decomposition.PLSRegression`)
- Model persistence: Pickle or joblib serialization
- Retraining: Monthly or when drift detected

#### 3.3 Output Generator

**Calculations**:
- **Pol %**: Primary output from PLS model
- **Moisture %**: Secondary PLS model or ratio-based calculation
- **Quality Flags**: Rule-based classification
  - "Excellent": Pol > 16%
  - "Good": Pol 14-16%
  - "Fair": Pol 12-14%
  - "Poor": Pol < 12%
- **Confidence Score**: Based on spectral quality and model uncertainty

**Output Format** (JSON):
```json
{
  "timestamp": "2026-02-25T10:30:45.123Z",
  "mill_id": "mill_001",
  "sensor_id": "sensor_01",
  "pol_percent": 14.2,
  "moisture_percent": 68.5,
  "quality": "good",
  "confidence": 0.95,
  "sensor_health": {
    "air_knife": "ok",
    "optical_window": "clear",
    "temperature": 35.2
  }
}
```

---

### Layer 4: Cloud & Storage Layer

**Purpose**: Route real-time data to clients and persist historical data for analytics.

#### 4.1 Realtime Router (Backend Server)

**Technology Options**:
- **FastAPI** (Python): High-performance async framework
- **Node.js + Express**: JavaScript ecosystem, excellent WebSocket support

**Responsibilities**:
- WebSocket server for real-time client connections
- MQTT broker or client (bridge from edge devices)
- REST API endpoints for historical queries
- Data validation and sanitization
- Load balancing for multiple sensors
- Authentication and authorization

**API Endpoints**:
```
GET  /api/sensors                    # List all sensors
GET  /api/sensors/{id}/current       # Current reading
GET  /api/sensors/{id}/history       # Historical data
POST /api/sensors/{id}/calibrate     # Trigger calibration
GET  /api/alerts                     # Alert history
POST /api/alerts/configure           # Set thresholds
```

**WebSocket Events**:
```
Client → Server:
  - subscribe: {sensor_id}
  - unsubscribe: {sensor_id}

Server → Client:
  - data: {pol, moisture, quality, ...}
  - alert: {type, message, severity}
  - health: {sensor_id, status}
```

#### 4.2 NoSQL Database

**Technology Options**:
- **Firebase Realtime Database**: Managed, real-time sync
- **MongoDB**: Flexible schema, excellent for time-series

**Collections/Tables**:
- `readings`: Time-series data (Pol %, moisture %, etc.)
- `sensors`: Sensor metadata and configuration
- `calibrations`: Calibration history and coefficients
- `alerts`: Alert history and acknowledgments
- `users`: User accounts and permissions
- `shifts`: Shift reports and summaries

**Data Retention**:
- Real-time data: 7 days (high resolution)
- Aggregated data: 1 year (hourly averages)
- Shift reports: Indefinite
- Calibration logs: Indefinite

#### 4.3 ERP Integration

**Purpose**: Export data to existing mill management systems.

**Integration Methods**:
- REST API for pull-based integration
- Webhook for push-based integration
- CSV/Excel export for manual import
- Database replication for direct access

**Exported Data**:
- Shift summaries (average Pol %, total tonnage)
- Quality trends
- Downtime reports
- Calibration status

---

### Layer 5: Client & Presentation Layer

**Purpose**: Provide operators with real-time insights and control.

#### 5.1 React Control Room Dashboard

**Technology Stack**:
- **Framework**: React 18+ with hooks
- **State Management**: Context API or Redux
- **Charting**: Recharts or Chart.js
- **WebSocket**: Native WebSocket API or Socket.io client
- **Styling**: Tailwind CSS or Material-UI
- **Build**: Vite or Create React App

**UI Components**:

**5.1.1 Real-Time Telemetry Display**
- Live line charts with rolling time window
- Pol % trend (primary metric)
- Moisture % trend (secondary metric)
- Quality score indicator (color-coded)
- Time range selector (5 min, 1 hour, shift)

**5.1.2 Alert & Notification System**
- Threshold monitoring (configurable limits)
- Visual alerts (flashing red/yellow/green)
- Audio alerts (optional, browser-based)
- Alert history log with timestamps
- Acknowledgment mechanism

**5.1.3 Sensor Health Monitor**
- Air-knife status (OK / Warning / Error)
- Optical window clarity (Clean / Dirty)
- Network connectivity (Connected / Disconnected)
- Calibration status (Valid / Expired)
- Temperature monitoring

**5.1.4 Historical Analytics**
- Shift reports (summary statistics)
- Batch comparisons (quality trends)
- Trend analysis (weekly/monthly)
- Export to CSV/PDF for reporting

**Responsive Design**:
- Desktop: Full dashboard with all components
- Tablet: Simplified view with key metrics
- Mobile: Alert notifications only

---

## Data Flow Architecture

### Real-Time Data Flow
```
Cane → Sensor → ESP32 → MQTT → Backend → WebSocket → Dashboard
  (Physical)  (I2C)   (WiFi)  (Processing) (Stream)   (Display)
  
Latency Breakdown:
  Sensor sampling:     200ms
  ESP32 processing:     50ms
  Network transmission: 50ms
  Backend processing:  100ms
  WebSocket delivery:   50ms
  UI rendering:         50ms
  ─────────────────────────
  Total:              ~500ms
```

### Historical Data Flow
```
Backend → Database → Query → API → Dashboard
         (Store)   (Retrieve) (REST) (Display)
```

### Calibration Flow
```
Reference Sample → Sensor → ESP32 → Backend → PLS Training → Model Update
                 (Scan)   (Data)  (Collect)  (Fit)        (Deploy)
```

---

## Network Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      MILL NETWORK                            │
│                                                              │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │ Sensor 1     │    │ Sensor 2     │    │ Sensor N     │  │
│  │ (ESP32)      │    │ (ESP32)      │    │ (ESP32)      │  │
│  └──────┬───────┘    └──────┬───────┘    └──────┬───────┘  │
│         │                   │                   │           │
│         └───────────────────┴───────────────────┘           │
│                             │                               │
│                    ┌────────▼────────┐                      │
│                    │  WiFi Router    │                      │
│                    │  / Access Point │                      │
│                    └────────┬────────┘                      │
│                             │                               │
│                    ┌────────▼────────┐                      │
│                    │  Edge Server    │                      │
│                    │  (Python AI)    │                      │
│                    └────────┬────────┘                      │
└─────────────────────────────┼───────────────────────────────┘
                              │ Internet / VPN
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                      CLOUD INFRASTRUCTURE                    │
│                                                              │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │ Backend      │◄──►│ Database     │◄──►│ ERP System   │  │
│  │ Server       │    │ (NoSQL)      │    │              │  │
│  └──────┬───────┘    └──────────────┘    └──────────────┘  │
│         │                                                   │
│         │ WebSocket                                         │
│         ▼                                                   │
│  ┌──────────────┐    ┌──────────────┐                      │
│  │ Dashboard 1  │    │ Dashboard N  │                      │
│  │ (Browser)    │    │ (Browser)    │                      │
│  └──────────────┘    └──────────────┘                      │
└─────────────────────────────────────────────────────────────┘
```

---

## Security Architecture

### Device Security
- **Firmware**: Signed OTA updates only
- **WiFi**: WPA2-Enterprise or WPA3
- **MQTT**: TLS encryption, username/password auth
- **Physical**: Tamper-evident enclosure seals

### Backend Security
- **API**: JWT-based authentication
- **WebSocket**: Token-based authorization
- **Database**: Role-based access control (RBAC)
- **Network**: Firewall rules, VPN for remote access

### Data Security
- **In Transit**: TLS 1.3 encryption
- **At Rest**: Database encryption
- **Backup**: Automated daily backups
- **Audit**: Logging of all access and changes

---

## Scalability Considerations

### Horizontal Scaling
- Multiple sensors per mill (load balanced)
- Multiple mills per backend instance
- Database sharding by mill_id or time range

### Vertical Scaling
- Backend server: CPU/RAM upgrades for more sensors
- Database: Storage expansion for longer retention

### Performance Optimization
- Edge processing reduces cloud load
- WebSocket connection pooling
- Database indexing on timestamp and sensor_id
- CDN for dashboard static assets

---

**Document Version**: 1.0.0  
**Last Updated**: February 2026  
**Related Documents**: [README.md](./README.md) | [process-flow-diagram.md](./process-flow-diagram.md) | [system-design.md](./system-design.md)
