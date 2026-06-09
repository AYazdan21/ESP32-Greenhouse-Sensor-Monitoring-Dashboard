````md
# ESP32 Greenhouse Sensor Monitoring Dashboard

A production-grade IoT monitoring and optimization platform for smart agricultural greenhouses. This repository hosts a dual-sensor telemetry framework powered by an ESP32 edge deployment paired with a containerized ThingsBoard data orchestrator. It couples real-time environmental processing loops with independent historical analytical overlays.

## 🚀 Key Features

- **Asynchronous Time-Window Engines**  
  Employs widget-level time window overrides. Visualizes real-time status gauges alongside historical table calculations without locking telemetry threads.

- **Edge Multi-Sensor Telemetry**  
  Processes incoming environmental matrix streams (Temperature and Humidity) from multiple spatial zones (`Greenhouse_Sensor_A` and `Greenhouse_Sensor_B`).

- **Custom Cold-Path Time-Series Widget**  
  Native HTML/JS runtime component executing statistical algorithms inside the dashboard engine over an isolated 1-hour historical sliding scale.

- **Automated Backend Analytics**  
  Dynamic matrix processing that handles absolute deltas and statistical deviations cleanly.

- **Native DevOps Integration**  
  Fully configured for ThingsBoard Version Control Services (VCS) to allow containerized Git workflows, version rollbacks, and reproducible clustering.

## 📊 Analytics Framework

The custom analytical pipeline computes performance parameters based on point-by-point comparative timestamps across your physical zones.

### Statistical Metrics Calculated

- **Maximum Temperature Delta**: Detects peak thermodynamic divergence between zones across the 1-hour frame.
- **Minimum Temperature Delta**: Confirms convergence efficiency and localized microclimatic stabilization.
- **95th Percentile Variance**: Filters out sensor anomalies, line spikes, and transmission drops by tracking the exact 95% line threshold of absolute temperature deviations:

$$
\Delta_{\text{95th}} = \mathbf{P}_{95}\left(\left|T_A(t) - T_B(t)\right|\right)
$$

## 📂 Repository Layout

ThingsBoard Version Control engine enforces a structured schema mapping directly to its internal schema database engine:

```text
├── dashboard/              # Compiled layout frameworks and entity view mappings
│   └── greenhouse_monitoring_system.json
├── rule_chain/             # Backend routing diagrams and data processing nodes
│   └── root_rule_chain.json
├── widget_bundle/          # UI component templates (contains the custom JS processors)
│   └── variance_analytics_bundle.json
├── device_profile/         # Device asset definitions and validation profiles
│   └── greenhouse_sensor_profile.json
└── README.md
````

## 🛠️ Installation & Microservice Setup

To import this configuration into your clean Dockerized ThingsBoard environment, follow this strict instantiation order to satisfy schema dependency paths:

### 1. Launch Your Dockerized Infrastructure

Verify that your local ThingsBoard containers are up and fully operational:

```bash
docker-compose up -d
```

### 2. Connect Your GitHub Repository Configuration

Log into your clean ThingsBoard console instance as a Tenant Administrator.

Navigate to **Advanced features → Version control** and fill out your Repository Settings:

* **Repository URL**: `https://github.com/AYazdan21/ESP32-Greenhouse-Sensor-Monitoring-Dashboard.git`
* **Default branch name**: `main`
* **Authentication method**: Password / access token
* **Access Token**: Paste your generated Personal Access Token (PAT) with repo scope permissions enabled.

Click **Check access** followed by **Save**.

### 3. Import Order Sequence

To prevent missing schema reference pointers, deploy or pull the entities from your local environment in this exact order:

1. **Widget Bundles**
2. **Device Profiles**
3. **Rule Chains**
4. **Dashboards**

#### Widget Bundles

Restores the custom HTML/CSS canvas rendering engine and structural logic blocks.

#### Device Profiles

Defines telemetry boundaries and registers expected payload variables (temperature, humidity).

#### Rule Chains

Boots the telemetry streaming logic nodes and connects the pipeline routing.

#### Dashboards

Connects your entity aliases and maps visual components to your active hardware data streams.

## 💻 Custom Time-Series Processing Engine

The analytics card runs entirely decoupled from the global real-time dashboards via a decoupled data loop hook inside `self.onDataUpdated`. Below is the clean algorithmic processing structure utilized:

```javascript
self.onDataUpdated = function() {
    var tbData = self.ctx.data;
    var container = self.ctx.$container[0];
    
    var maxEl = container.querySelector('#max-delta');
    var minEl = container.querySelector('#min-delta');
    var p95El = container.querySelector('#p95-delta');

    if (!tbData || tbData.length < 2) return;

    var dataA = tbData[0].data;
    var dataB = tbData[1].data;

    if (!dataA || !dataB || dataA.length === 0 || dataB.length === 0) return;

    var deltas = [];
    var len = Math.min(dataA.length, dataB.length);

    // Compute point-by-point absolute array distances
    for (var i = 0; i < len; i++) {
        deltas.push(Math.abs(dataA[i][1] - dataB[i][1]));
    }

    if (deltas.length > 0) {
        var maxDelta = Math.max.apply(null, deltas).toFixed(1);
        var minDelta = Math.min.apply(null, deltas).toFixed(1);
        
        // Isolate the 95th Percentile Variance via ascending quicksort
        deltas.sort(function(a, b) { return a - b; });
        var percentileIndex = Math.floor(0.95 * deltas.length);
        var p95Delta = deltas[percentileIndex].toFixed(1);

        maxEl.innerHTML = maxDelta + " °C";
        minEl.innerHTML = minDelta + " °C";
        p95El.innerHTML = p95Delta + " °C";
    }
};
```

## 🎛️ Hardware Interface Configuration

For physical edge hardware tracking, configure your ESP32 microcontrollers to target your ThingsBoard server instance using standard MQTT communication ports:

* **Broker Port**: `1883` (default non-encrypted) or `8883` (secure MQTT over TLS)
* **Topic Scheme**: `v1/devices/me/telemetry`

### Payload Schema Format

```json
{
  "temperature": 24.5,
  "humidity": 58.2
}
```

## 📄 License

This project is licensed under the MIT License - see the `LICENSE` file for complete compliance details.

