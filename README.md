# Icinga2 Check Plugin for Aruba 6000 Switches

This Python script is an Icinga2 check plugin to monitor **Aruba CX/6000 series switches** via SNMP. It checks system uptime, temperature, interface status/bandwidth, PSU status, and fan status.

---

## 📋 Overview

The plugin retrieves critical health metrics from an Aruba switch using SNMP and reports them to Icinga2. It returns:
- **OK**, **WARNING**, **CRITICAL**, or **UNKNOWN** statuses.
- Detailed output for troubleshooting.

### Checks Performed
- **Temperature** (with custom thresholds)
- **Uptime**
- **Interface status** (up/down) and bandwidth usage
- **Power Supply (PSU) status**
- **Fan status**

---

## ⚙️ Requirements

- Python 3.x
- `pysnmp` library (IMPORTANT!!! INSTALL PYSNMP 4.4.X!!! IF YOU INSTALL THE NEWEST VERSION IT MIGHT NOT WORK DUE TO MISSING DEPENDENCIES)
- SNMP enabled on the Aruba switch (community string configured)

---

## 📥 Installation

1. **Install the `pysnmp` library**:
   ```bash
   pip install pysnmp
   ```

2. **Save the script** as `check_aruba6000.py` and make it executable:
   ```bash
   chmod +x check_aruba6000.py
   ```

3. **Copy the script** to your Icinga2 plugins directory (e.g., `/usr/lib/nagios/plugins/`).

---

## 🛠️ Usage

### Command-Line Arguments

| Argument          | Description                                   | Default     |
|-------------------|-----------------------------------------------|-------------|
| `--host`          | Switch IP/hostname (required)                | -           |
| `--community`     | SNMP community string                        | `public`    |
| `--port`          | SNMP port                                    | `161`       |
| `--warn-temp`     | Temperature warning threshold (°C)           | `50`        |
| `--crit-temp`     | Temperature critical threshold (°C)          | `70`        |

### Example

```bash
./check_aruba6000.py \
  --host 192.168.1.1 \
  --community private \
  --warn-temp 55 \
  --crit-temp 75
```

### Output Example
```
Interfaces up: 8/24
Uptime: 14d 3h 22m 10s
Temperature: 45.32°C
IF1 (GigabitEthernet1/0/1): up, In: 1.23 GB, Out: 4.56 GB
IF2 (GigabitEthernet1/0/2): down, In: 0.00 B, Out: 0.00 B
PSU 1: OK
PSU 2: FAIL (value: 0)
Fan 1: OK
```

---

## 🔢 Exit Codes

| Code | Status       | Description                              |
|------|--------------|------------------------------------------|
| 0    | OK           | All components are healthy.              |
| 1    | WARNING      | Temperature exceeds warning threshold.   |
| 2    | CRITICAL     | Temperature/PSU/Fan in critical state.   |
| 3    | UNKNOWN      | SNMP query failed or invalid data.       |

---

## 🔄 Icinga2 Integration

1. **Define a command** in Icinga2’s `commands.conf`:
   ```javascript
   object CheckCommand "check_aruba6000" {
     command = [ PluginDir + "/check_aruba6000.py" ]
     arguments = {
       "--host" = "$address$"
       "--community" = "$snmp_community$"
       "--warn-temp" = "$aruba_warn_temp$"
       "--crit-temp" = "$aruba_crit_temp$"
     }
   }
   ```

2. **Create a service check** in `services.conf`:
   ```javascript
   apply Service "Aruba 6000 Health" {
     check_command = "check_aruba6000"
     vars.snmp_community = "public"
     vars.aruba_warn_temp = 50
     vars.aruba_crit_temp = 70
     assign where host.address && host.vendor == "Aruba"
   }
   ```

---

## 📝 Notes

- Ensure SNMP is enabled on the Aruba switch with the correct community string.
- Adjust temperature thresholds (`--warn-temp`/`--crit-temp`) as needed.
- Test the script manually before integrating with Icinga2.

**Author**: deltaseconds (me) 
**License**: MIT License
