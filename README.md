# Spoolman Syncer

**Spoolman Syncer** is an advanced automation wrapper for the [spoolman2slicer](https://github.com/bofh69/spoolman2slicer) utility. It bridges the gap between your **Spoolman** inventory and **OrcaSlicer/BambuStudio**, providing a "zero-configuration" experience with enhanced Klipper integration.

## 🖨️ Compatibility & Roadmap

* **Principal Use Case**: Currently validated and optimized for the **Anycubic Kobra S1**.
* **Roadmap**: Specific configurations for **Bambu Lab P2S** and **H2D** are planned for upcoming releases.

## ✨ Key Features

* **Autonomous Environment Management**:
  * **Zero Dependencies**: You only need Python installed. The script automatically fetches the generator tool, creates an isolated `.venv`, and installs requirements.
  * **Self-Healing**: Automatically detects if dependencies are missing or outdated and updates the environment without user intervention.
* **Intelligent Post-Processing**:
  * **Crash Prevention**: Robustly handles special characters in Spoolman names (e.g., `PLA+/Pro`) that normally cause generator crashes.
  * **Material Mapping**: Automatically corrects JSON inheritance (e.g., mapping custom "PLA+/Pro" types to standard "Generic PLA") to prevent Slicer import errors.
  * **Klipper ID Injection**: Injects `M555 S={ID}` into the start G-code, enabling precise spool tracking in Klipper macros.
* **Deployment Workflow**:
  * Supports automated installation to Slicer directories.
  * Includes cleaning utilities to purge old configurations.

## 🚀 Prerequisites

* **Python 3.x** (Compatible with `python`, `python3`, or `py` commands).
* A running instance of [Spoolman](https://github.com/Donkie/Spoolman).

## 📥 Installation

1.  Download `spoolman_syncer.py`.
2.  Place it in a dedicated directory (e.g., `~/3dprint-tools/`).

## 🛠️ Usage Guide

### 1. Basic Generation
Fetches data and generates profiles in the local `spools/` folder. Safe to run for testing.
```bash
python spoolman_syncer.py --ip 192.168.1.50 --port 7912
```

### 2. Install to Slicer
Generates profiles and installs them directly to the Slicer's user directory.
```bash
python spoolman_syncer.py --ip 192.168.1.50 --apply orca
# OR
python spoolman_syncer.py --ip 192.168.1.50 --apply bambu
```

### 3. Clean Install (Recommended)
Use `--delete-first` to wipe the Slicer's filament folder before installing. This ensures spools deleted from Spoolman are removed from the Slicer.
```bash
python spoolman_syncer.py --ip 192.168.1.50 --apply orca --delete-first
```

### 4. Maintenance Commands
* `--clean-spool`: Deletes the local `spools/` output folder before running.
* `--clean`: **Factory Reset**. Deletes the tool source, the virtual environment, and output files. Use this if the script behaves unexpectedly or you want to reinstall the tool.

### Argument Reference

| Argument | Default | Description |
| :--- | :--- | :--- |
| `--ip` | `localhost` | Spoolman Server IP address. |
| `--port` | `7912` | Spoolman Server Port. |
| `--apply` | `None` | Target Slicer (`orca` or `bambu`). |
| `--delete-first` | `False` | Wipes target Slicer directory before copying. |
| `--clean-spool` | `False` | Clears local generated files before execution. |
| `--clean` | `False` | Deletes all tool/env folders and exits. |

---

## ⚙️ Klipper Integration

To utilize the injected `M555 S={ID}` command, add the following macros to your Klipper `printer.cfg`.

> **⚠️ Hardware Note:** Validated for **Anycubic Kobra S1**. Bambu Lab integration pending.

```ini
[gcode_macro M555]
description: Sets the active Spoolman Spool ID
gcode:
    {% if params.S is defined %}
        SET_SPOOL SPOOL_ID={params.S}
    {% endif %}

[gcode_macro SET_SPOOL]
description: Calls the remote Spoolman method
gcode:
    {% if params.SPOOL_ID is defined %}
        {action_call_remote_method(
            "spoolman_set_active_spool",
            spool_id=params.SPOOL_ID|int
        )}
    {% endif %}
```

*Note: Ensure `[spoolman]` is configured in your `moonraker.conf`.*

## 🏆 Credits

* **Author**: Nebulino
* **Core Generator**: Based on [bofh69/spoolman2slicer](https://github.com/bofh69/spoolman2slicer).