# 🔋 Battery Optimizer Light Plus (Home Assistant Integration)

**Battery Optimizer Light Plus** is an advanced, high-performance integration designed to optimize battery usage, dynamic energy tariff management, and peak shaving for home energy storage systems (including Sonnen, Victron, SolarEdge, and other supported batteries). 

Building upon the core features of Battery Optimizer Light, the **Plus** edition introduces multi-battery management, enhanced spot-price arbitrage algorithms, fine-grained solar production forecasting, and extended local automation controls.

---

## ✨ Features

* **📈 Advanced Spot Price Arbitrage:** Dynamic charging/discharging based on real-time Nord Pool, Tibber, or ENTSO-E spot market prices.
* **⚡ Smart Peak Shaving & Load Balancing:** Local, low-latency monitoring to prevent tripping main breakers or exceeding grid capacity tariffs.
  * **Dynamic Thresholds:** Automatically adjusts peak power limits based on hourly grid tariff tiers.
  * **Hysteresis Guard:** Prevents rapid toggling and flutter during load fluctuations.
* **☀️ Solar Production & Consumption Forecasting:** Integrates with Solcast or Forecast.Solar to predict daily energy generation and tailor charging schedules accordingly.
* **🔋 Multi-Battery & Hybrid Support:** Manage multiple battery storage systems or hybrid inverter setups under a unified optimization engine.
* **⛄ Winter & Emergency Reserves:** Configurable State-of-Charge (SoC) floor to reserve emergency backup power during high-risk seasons or outages.
* **📊 Dashboard & Metrics:** Directly streams performance analytics, daily cost savings, and top power peaks to your Home Assistant dashboard and cloud web portal.

---

## 🛠️ Prerequisites

1. **Home Assistant** version 2023.8 or higher.
2. A supported battery or inverter integration (e.g., Sonnen, Victron, Modbus/REST control).
3. An active API key from the **Battery Optimizer Light** dashboard/service.
4. (Optional) A solar forecast integration (Solcast or Forecast.Solar) installed in Home Assistant.

---

## 🚀 Installation

### Option A: HACS (Recommended)

1. Open **HACS** in Home Assistant → **Integrations**.
2. Click the three dots (top-right corner) → **Custom repositories**.
3. Add Repository URL: `https://github.com/awestin67/battery-optimizer-light-plus`
4. Set Category to **Integration**.
5. Click **Add**, search for **Battery Optimizer Light Plus**, and click **Download**.
6. Restart Home Assistant.

### Option B: Manual Installation

1. Download the latest release source code.
2. Copy the `battery_optimizer_light_plus` folder to `/config/custom_components/`.
3. Restart Home Assistant.

---

## ⚙️ Configuration

1. In Home Assistant, go to **Settings** → **Devices & Services**.
2. Click **+ Add Integration** and search for **Battery Optimizer Light Plus**.
3. Complete the setup wizard:
   * **API Key:** Your authentication key from the optimization cloud platform.
   * **Battery SoC Sensor:** Sensor tracking current battery level (`%`).
   * **Grid Power Sensor:** Sensor measuring grid import/export (`W` or `kW`).
   * **Battery Power Sensor:** Sensor measuring active battery charge/discharge (`W` or `kW`).
   * **Solar Forecast Sensor:** *(Optional)* Sensor predicting remaining daily solar generation (`kWh`).
   * **Consumption Forecast Sensor:** *(Optional)* Sensor predicting remaining daily load (`kWh`).
   * **Min/Max Reserve SoC:** Set minimum reserve floors and maximum target limits.

---

## ℹ️ Exposed Sensors & Controls

The integration creates the following entities in Home Assistant:

* `sensor.optimizer_plus_action`: Active decision (`CHARGE`, `DISCHARGE`, `HOLD`, `IDLE`).
* `sensor.optimizer_plus_charge_target`: Target charge power setpoint (`W`).
* `sensor.optimizer_plus_discharge_target`: Target discharge power setpoint (`W`).
* `sensor.optimizer_plus_peak_limit`: Active grid limit threshold (`W`).
* `sensor.optimizer_plus_estimated_savings`: Real-time daily cost savings calculation.
* `switch.optimizer_plus_peak_shaving`: Toggle to enable/disable local peak shaving on the fly.
* `switch.optimizer_plus_force_charge`: Manual override to initiate immediate emergency charging.

---

## 🤖 Automation Integration (YAML Example)

Use this base automation to trigger your battery inverter whenever the optimization engine issues a new command:

```yaml
alias: "🔋 Battery Optimizer Light Plus - Local Execution"
description: "Applies power setpoints to the battery based on Optimizer Plus actions."
triggers:
  - trigger: state
    entity_id: sensor.optimizer_plus_action
  - trigger: time_pattern
    minutes: "/5"
conditions:
  - condition: not
    conditions:
      - condition: state
        entity_id: sensor.optimizer_plus_action
        state:
          - unknown
          - unavailable
actions:
  - variables:
      action: "{{ states('sensor.optimizer_plus_action') }}"
      charge_w: "{{ states('sensor.optimizer_plus_charge_target') | float(0) }}"
      discharge_w: "{{ states('sensor.optimizer_plus_discharge_target') | float(0) }}"
  - choose:
      - conditions:
          - condition: template
            value_template: "{{ action == 'CHARGE' }}"
        sequence:
          - action: script.battery_set_charge_power
            data:
              power: "{{ charge_w }}"
      - conditions:
          - condition: template
            value_template: "{{ action == 'DISCHARGE' }}"
        sequence:
          - action: script.battery_set_discharge_power
            data:
              power: "{{ discharge_w }}"
      - conditions:
          - condition: template
            value_template: "{{ action == 'IDLE' }}"
        sequence:
          - action: script.battery_set_auto_mode
mode: single
