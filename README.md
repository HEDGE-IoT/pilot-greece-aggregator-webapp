# Web App for Aggregator

## 1.1 General Information and Purpose
- **Service Name/Title:** Web App for Aggregator  
- **Description and purpose:**  
The Aggregator Web App is a centralised platform for energy data monitoring, analytics, and forecasting. It aggregates data from IoT-enabled Distributed Energy Resources (DERs) and provides real-time and historical insights into energy consumption and production. The platform supports model training, forecasting, and anomaly detection for smart grid applications.  
- **Owner/Contact Information:** ICCS

---

## 1.2 Functional Requirements
- Aggregate real-time and historical energy consumption and production data from multiple households and devices  
- Train forecasting models (short-term and 24-hour) based on historical data  
- Perform recursive and non-recursive forecasts  
- Display consumption, production, and net energy balance via dashboards  
- Provide detailed analytics including anomalies, voltage events, power factor trends, and load profiles  
- Export summary statistics (daily, weekly, monthly) for further analysis  
- Role-based access to dashboard views and APIs  

---

## 1.3 Non-Functional Requirements

### Performance
- Real-time data ingestion latency < 2s  
- Forecast generation < 5s per request  

### Reliability and Availability
- Uptime > 99.5%  
- Failover and retry mechanisms for model training and forecasts  

### Security
- Authentication via JWT tokens  
- Role-based authorisation for endpoints and UI views  
- Data encryption in transit (TLS 1.2+) and at rest (AES-256)  
- GDPR-aligned data handling and anonymisation practices  

---

## 1.4 Service Interfaces

### 1.4.1 API Endpoints

#### Endpoint 1 — Train Consumption Model for Next 24 Hours Predictions
- **URL:** `/aggregator/consumption/train/next-24-hours`  
- **Method:** `POST`  
- **Description:** Trains a sequence-to-sequence model using aggregated hourly energy usage data and saves it to MinIO.  

**Response Example**
```json
{
  "message": "Model trained and saved to MinIO successfully",
  "mse": 0.0342,
  "mae": 0.1551,
  "r2": 0.8912
}
```

**Error Handling**
```json
500 Internal Server Error: { "error": "Training failed" }
```

---

#### Endpoint 2 — Train Consumption Model for Short Term Predictions (15 min)
- **URL:** `/aggregator/consumption/train/short-term`  
- **Method:** `POST`  
- **Description:** Trains a GRU model for forecasting up to 5 hours in 15-minute steps.  

**Example Request**
```
POST /aggregator/consumption/train/short-term?horizon=8
```

**Response Example**
```json
{
  "message": "Model trained and saved to MinIO successfully",
  "mse": 0.02,
  "mae": 0.12,
  "r2": 0.89,
  "horizon": 8
}
```

**Error Handling**
```json
500 Internal Server Error: { "error": "Training failed" }
422 Validation Error: { "error": "Invalid or missing input" }
```

---

#### Endpoint 3 — Get Forecast (Next 24 Hours)
- **URL:** `/aggregator/consumption/forecast/next-24-hours`  
- **Method:** `POST`  
- **Description:** Returns 24-hour forecast using pre-trained LSTM models.  

**Response Example**
```json
{
  "forecast": [
    { "timestamp": "2025-07-17 01:00:00", "forecasted_energy_usage": 215.47 }
  ]
}
```

**Error Handling**
```json
500 Internal Server Error: { "error": "Model missing or forecast failed" }
```

---

### 1.4.2 UI Mock-ups
- Overview, Real-Time Monitoring, Historical Statistics, Forecasts — included in Section 1.8.  

---

## 1.5 Data Model

### Entities and Relationships
- **Users ↔ Houses:** Many-to-many.  
- **Houses ↔ Devices:** One-to-many. Devices may be appliances or energy meters.  
- **Users ↔ Energy Communities:** Many-to-many.  
- **PV Parks ↔ Energy Communities:** Community-level solar installations.  
- **PVs ↔ Houses:** Houses may host individual PVs.  

### Database Schema
- **Real-Time Energy Measurements:** Captures voltage, current, power, energy (high frequency).  
- **Aggregated Energy Usage:** Summarizes hourly/daily/quarterly consumption and production.  
- **Metadata Schema:** Tracks users, houses, devices, energy communities, and PVs.  

---

## 1.6 Integration and Dependencies

### External Dependencies
- Weather APIs  
- MinIO Object Storage  

### System Dependencies
- PostgreSQL  
- FastAPI  
- Redis (job queue) + Celery  

### Third-party Integrations
- MinIO for model storage  
- Grafana (optional visualization)  

### HEDGE-IoT Integration
- Ingests data from IoT-enabled DERs  
- Sync with cloud APIs  

### App Store Integration
- Packaged APIs compatible with deployment via containerization  

### Data Space Integration
- Data access via federated APIs, GDPR-compliant interfaces  

---

## 1.7 Security and Privacy

### Data Sensitivity
- Personal identifiable information minimized  
- Energy usage data treated as sensitive operational data  

### Access Control
- Role-based access: admin, analyst, consumer, third-party app  
- Token-based API access  

### Audit Logs
- API access logs stored and rotated  
- Model training + forecast logs retained for 90 days  

---

### Aggregator – Overview
Displays total demand, total production, net energy balance, and user distribution (consumers, prosumers, community members).  

### Real-Time Monitoring
Line charts for real-time consumption and production.  

### Historical Statistics
Bar charts for consumption and production, with toggles for 24h/7d data. Summary panels with averages and maximums.  

### Forecasts
Consumption and production forecast charts, split between historical (blue) and forecast (red) with confidence range (orange).  

---
