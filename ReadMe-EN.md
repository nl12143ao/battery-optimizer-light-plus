# Battery Optimizer Light Plus (BOL+)

Optimization script for home batteries via Home Assistant. Works with Home Assistant version **2024.11** or later.

> **Tip:** If you only need energy optimization, check out **[Energy Optimizer](https://github.com/nl12143ao/energy-optimizer)**!

---

## 🚀 Key Features

* **Tibber Integration:** Automatically retrieves electricity prices from Tibber (supports heating control, EV charging, etc.).
* **Smart Solar Forecast:** Fetches solar power forecasts from **Forecast.Solar**.
* **Automatic Charge & Discharge:** Optimizes charging and discharging cycles to maximize savings.
* **Flexible Operating Modes:**
  * **Auto:** Fully automated optimization.
  * **Optimized Manual:** Manual adjustments combined with auto-optimization.
  * **Full Manual:** Manual control over battery behavior.
* **Safety Rules & Emergency Stop:** Built-in safeguards against grid overload and rapid battery degradation.

---

## 🛠️ Prerequisites

Before you start, make sure you have:

* **Home Assistant** (version **2024.11** or later).
* The **Tibber Integration** installed and configured in Home Assistant.
* The **Forecast.Solar Integration** configured.
* Access to **Developer Tools** in Home Assistant to create helper entities (*input_number*, *input_select*, *input_boolean*, etc.).

---

## 📦 Installation & Setup

1. **Download the Script:**
   * Download `battery_optimizer.py` from this repository and place it in your `python_scripts/` directory in Home Assistant.
2. **Create Required Helper Entities:**
   * Go to **Settings → Devices & Services → Helpers**.
   * Create the entities listed below (see [Helper Entities](#-helper-entities)).
3. **Configure Automation:**
   * Create a new automation in Home Assistant that runs the script every hour or when electricity prices update.

---

## ⚙️ Helper Entities

Create the following entities in Home Assistant:

| Name (Entity ID) | Type | Description |
| :--- | :--- | :--- |
| `input_select.battery_mode` | Dropdown | Select mode: `Auto`, `Optimized Manual`, `Full Manual`. |
| `input_number.battery_min_soc` | Number | Minimum allowed State of Charge (SoC) (%). |
| `input_number.battery_max_soc` | Number | Maximum allowed State of Charge (SoC) (%). |
| `input_number.target_charge_kwh` | Number | Targeted charging amount (kWh). |

*(Add more helper entities to the table if needed based on your specific setup).*

---

## 📄 License

Distributed under the **MIT License**. See `LICENSE` for more information.
