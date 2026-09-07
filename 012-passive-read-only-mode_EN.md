# ADR 012: Passive "Read-Only" Mode for Home Assistant

**Date:** 2026-05-15  
**Status:** Accepted

## Context

According to ADR 007, we introduced a "Split-Brain" protection mechanism that completely blocks Home Assistant (V1 `/signal`) if the user has selected the Edge dongle as the active control unit. Currently, the backend returns hardcoded zero-values and `action="IDLE"` to HA when this occurs.

The issue is that many users have built extensive dashboards in Home Assistant and want to continue seeing what the system is doing (current action, target power, next action, baseload, etc.) even if it is the Edge dongle executing the actual Modbus control.

Furthermore, the HA integration currently features a built-in safety mechanism that automatically sends the command `IDLE` (Automatic Self-consumption) to the inverter if it loses contact with the cloud. If Edge is the active control unit, HA must absolutely not intervene and reset the battery during a network outage.

## Decision

We are converting the Home Assistant integration's role to act as "Read-Only" (Passive) when Edge is selected as the primary client.

### 1. Backend: Modification of Split-Brain Protection in `/signal` (V1)

Instead of strictly rejecting HA with zeroed-out dummy values, `/signal` will return the actual state, but mark the response as passive and avoid writing history to the database.

* **New API Field:** We add `client_mode: str = "ACTIVE"` (default) to `SignalResponse` (V1).
* **Passive Logic:** If `active_client_type == 'Edge'` when HA calls `/signal`:
  * Set `client_mode = "PASSIVE"`.
  * Retrieve the latest optimization decision from `LAST_OPTIMIZATION_RESULT_CACHE` (which the Edge dongle continuously updates) or run a "Dry-Run" of the optimizer if the cache is empty.
  * **Important:** Skip the call to `db.log_decision` to avoid HA creating duplicate rows in `light_decision_logs`. Duplicate logs would distort AI learning, efficiency calculations, and savings analytics.

### 2. HA Integration: Passive Mode and Fallback Protection

The Home Assistant integration (the Python code in the `battery-optimizer-light-plus` repository) is updated to read and respect the new `client_mode` field.

* **Modbus Blocking:** If the integration receives `client_mode == "PASSIVE"`, it must only update its sensors in HA (so that dashboards function). It must **not** execute any Modbus writes (`CHARGE`, `DISCHARGE`, `HOLD`, `IDLE`).
* **Deactivation of Fallback:** If the integration is in passive mode and loses connection to the cloud (e.g., timeout), it must **not** send the fallback command `IDLE` to the inverter. It remains silent and allows the Edge dongle to handle all error management.

## Consequences

* **Seamless Dashboard Experience:** Users can switch to the Edge dongle without their existing HA dashboards breaking. Sensors continue to display exactly what the optimizer (via Edge) is doing.
* **Protection Against Conflicts:** The dangerous conflict where HA loses internet connectivity and forces the inverter to `IDLE` while Edge is attempting to control it is completely eliminated.
* **Clean History:** By preventing HA from logging its "Read-Only" requests to the database, data accuracy in `light_decision_logs` is preserved.
* **Update Requirement:** For offline protection (Fallback) to function properly, users must update their HA integration to a version that supports `client_mode`.
