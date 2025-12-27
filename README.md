# Spoolman Syncer

**Spoolman Syncer** is an intelligent automation wrapper for the [spoolman2slicer](https://github.com/bofh69/spoolman2slicer) tool. It completely automates the process of syncing your **Spoolman** filament inventory to **OrcaSlicer** or **BambuStudio**, handling everything from environment setup to Klipper integration.

## 🖨️ Hardware & Roadmap

* **Principal Use Case**: This tool is currently optimized and tested principally for the **Anycubic Kobra S1**.
* **Future Updates**: I am planning to add specific configurations and optimizations for **Bambu Lab P2S** and **H2D** printers in upcoming releases, as I use these machines alongside the Kobra.

## ✨ Features

* **Zero-Config Installation**: You don't need to manually install the generator tool or Python libraries. The script automatically:
    * Downloads the generator repository if missing.
    * Creates a local isolated `venv` (virtual environment).
    * Installs all necessary dependencies.
    * **Smart Updates**: Automatically detects if dependencies change in future versions and updates the environment.
* **Smart Material Mapping**: Automatically fixes JSON inheritance issues (e.g., mapping "PLA+/Pro" to "Generic PLA" to prevent slicer errors).
* **Klipper ID Injection**: Injects a custom `M555 S={ID}` G-code command into every filament start G-code. This allows Klipper to automatically track exactly which spool you are using.
* **Auto-Deployment**: Can automatically copy the generated profiles directly to your slicer's configuration folder.

## 🚀 Prerequisites

* **Python 3.x** installed on your system.
* A running instance of [Spoolman](https://github.com/Donkie/Spoolman).

## 📥 Installation

1.  Download the `spoolman_syncer.py` script.
2.  Place it in a folder where you want to keep your filament profiles.

## 🛠️ Usage

Open your terminal or command prompt in the script's folder.

### Basic Run
Generates the JSON files in the `spools/` folder but does **not** install them to the slicer.
```bash
python spoolman_syncer.py --ip 192.168.1.50 --port 7912
```

### Install to OrcaSlicer
Generates files and immediately copies them to the OrcaSlicer user directory.
```bash
python spoolman_syncer.py --ip 192.168.1.50 --apply orca
```

### Install to BambuStudio
```bash
python spoolman_syncer.py --ip 192.168.1.50 --apply bambu
```

### Clean Install (Delete old files first)
Useful if you have removed spools from Spoolman and want them gone from the slicer too.
```bash
python spoolman_syncer.py --ip 192.168.1.50 --apply orca --delete-first
```

### Command Line Arguments

| Argument | Default | Description |
| :--- | :--- | :--- |
| `--ip` | `localhost` | The IP address of your Spoolman server. |
| `--port` | `7912` | The port of your Spoolman server. |
| `--apply` | `None` | Automatically install profiles. Options: `orca`, `bambu`. |
| `--delete-first` | `False` | Delete all existing JSON files in the slicer folder before copying new ones. |

---

## ⚙️ Klipper Configuration

This script injects `M555 S={ID}` into your filament start G-code. To make Klipper understand this command and notify Spoolman, add the following macros to your `printer.cfg`.

**⚠️ Note:** The configuration below is currently verified principally for the **Anycubic Kobra S1**. Configurations for other printers (like Bambu Lab) will be added in future updates.

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

*> **Note:** You must have Spoolman configured in your `moonraker.conf` for `action_call_remote_method` to work.*

## 📂 How it Works

1.  **Environment Check**: The script checks if the `spoolman2slicer_tool` folder and the local `.venv` exist.
2.  **Auto-Setup**: If missing, it downloads the source code from GitHub, extracts it, creates a virtual environment, and installs `requirements.txt`.
3.  **Generation**: It runs the generator inside the virtual environment to fetch data from Spoolman and saves it to the `spools/` folder.
4.  **Patching**: It iterates through every generated `.json` file:
    * Fixes "Inherits" fields to ensure compatibility with standard slicer profiles.
    * Extracts the `filament_settings_id` (Spool ID).
    * **Appends** `M555 S={ID}` to the start G-code (preserving existing commands like `ASSERT_ACTIVE_FILAMENT`).
5.  **Deployment**: If `--apply` is used, files are copied to your OS-specific Slicer configuration path.

## 🏆 Credits

* **Spoolman Syncer** created by **Nebulino**.
* Based on the generator logic from [bofh69/spoolman2slicer](https://github.com/bofh69/spoolman2slicer).