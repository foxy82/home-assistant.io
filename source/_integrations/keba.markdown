---
title: "Keba Charging Station"
description: "Instructions on how to setup your Keba charging station with Home Assistant."
logo: keba.png
ha_category:
  - Binary Sensor
  - Lock
  - Sensor
ha_release: 0.98
---

The `keba` integrates your Keba charging station (wallbox) into your home assistant instance. It was tested with a BMW Wallbox but should also work with a Keba P20/P30 according to the developers [manual](https://www.keba.com/web/downloads/e-mobility/KeContact_P20_P30_UDP_ProgrGuide_en.pdf). The fetching interval to the charging station is set to 5 seconds, same as in the official mobile app.

This component provides the following platforms:

- Binary Sensors: Online state, plug state, Charging state and failsafe mode state.
- Lock: Authorization (like with the RFID card).
- Sensors: current set by the user, target energy set by the user, charging power, charged energy of the current session and total energy charged.
- Services: authorize, deauthorize, set energy target, set maximum allowed current and manually update the states. More details can be found [here](/integrations/keba/#services).

## Configuration

To enable this component in your installation, add at least the following to your `configuration.yaml` file:

```yaml
# Example configuration.yaml entry
keba:
  host: KEBA_HOST
```

{% configuration %}
keba:
  description: configuration
  required: true
  type: map
  keys:
    host:
      description: Keba host.
      required: true
      type: string
    rfid:
      description: RFID tag used for authorization.
      required: false
      type: string
      default: "00845500"
    failsafe:
      description: Enable failsafe mode at home assistant startup.
      required: false
      type: boolean
      default: false
    failsafe_timeout:
      description: Timeout of the failsafe mode in seconds. Allowed values are between 10 seconds and 600 seconds (this parameter is only used if failsafe mode is enabled). Make sure to call the `keba.set_curr` service regularly within this timeout period!
      required: false
      type: integer
      default: 30
    failsafe_fallback:
      description: Fallback current of the failsafe mode in A. Allowed values are between 6 Ampere and 63 Ampere. 0 Ampere disables the running charging process completely (this parameter is only used if failsafe mode is enabled).
      required: false
      type: integer
      default: 6
    failsafe_persist:
      description: Saving the failsafe configuration to internal EEPROM of the Keba charging station. 1 means save it, 0 means do only keep this configuration until the next restart of the charging station (this parameter is only used if failsafe mode is enabled).
      required: false
      type: integer
      default: 0
    refresh_interval:
      description: Refresh interval to fetch new data from the charging station. 5 seconds (same as in the official app) is recommended.
      required: false
      type: integer
      default: 5
{% endconfiguration %}

## Services

The `keba` component offers several services. Using these services will change the state of your charging station. So use these services with care!

### Authorizing and Deauthorizing `keba.authorize` and `keba.deauthorize`

The charging station can be authorized and deauthorized via service calls (`keba.authorize` and `keba.deauthorize`) or via the lock component that is created automatically for the charging station. In both cases the RFID tag from the configuration is used.

### Start and Stop `keba.start` and `keba.stop`

The service `keba.start` and `keba.stop` controls the charging process if the car is already authorized. Technically it sends `ena 1` or `ena 0` commands to the charging station.

### Set Target Energy `keba.set_energy`

The service `keba.set_energy` sets the target energy for the current session to the given energy attribute in kWh. Payload example:

```json
{
  "energy": 10.0
}
```

### Maximum Current `keba.set_curr`

The service `keba.set_curr` sets the maximum current to the given current attribute in Ampere. Payload example:

```json
{
  "current": 16.0
}
```

### Request New Data `keba.request_data`

The service `keba.request_data` sends data update requests to the charging station.

### Request New Data `keba.set_failsafe`

The service `keba.set_failsafe` sets the failsafe mode of the charging station. Payload example:

```json
{
  "failsafe_timeout": 30,
  "failsafe_fallback": 6,
  "failsafe_persist": 0
}
```

## Disclaimer

This software is not affiliated with or endorsed by Keba.
