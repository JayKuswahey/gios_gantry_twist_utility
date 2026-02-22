# Gio's Gantry Twist Utility

Klipper module to visualize and adjust gantry twist for printers running the Beacon Eddy Current Surface Scanner

## Overview

This module has 2 modes: analysis and compensation.

Analysis mode samples the bed and generates 3 dashboards with various graphs to help diagnose gantry alignment issues by visualizing the Z-offset measurements from Beacon's contact and proximity modes. It leverages the `BEACON_OFFSET_COMPARE` command to calculate deltas across your print bed.

Ideally, the delta between contact and proximity measurements should be zero (or at least consistent) across the bed. Variations indicate that the Beacon probe and nozzle are operating on different planes, leading to probing inaccuracies and first-layer issues.

Compensation mode works out and applies the Z compensation values for Klipper's native gantry twist compensation module `[axis_twist_compensation]`.

### Requirements

- Klipper
- Beacon probe
- SSH access to your printer
- A printer with internet access (only if using the recommended installation script below)

### Compatibility

This module can work with any printer running Klipper but was designed and tested on a QIDI Plus 4. As such, the installation guide and the default settings are tailored for it. If you install/run on a different printer, please review the installation and settings carefully.

## Installation

Access your printer via SSH to execute the following parts

### Backup your configuration and klipper objects

On a command shell (`ssh`) to the printer, run the following

> [!CAUTION]
> Be sure to update the value for `reason`, or else you might be overwriting a previous backup.

```bash
date=$(date +'%Y%m%d')
reason='beforegiosgantrytwist'
mkdir -p /home/mks/qidi-klipper-backup-${date}-${reason}
(cd /home/mks; tar cvf - klipper printer_data/config) | (cd /home/mks/qidi-klipper-backup-${date}-${reason}; tar xf -)
```

### Install Gantry Twist Utility

 1. Install the python modules needed

```bash
cd /home/mks && wget -O - https://raw.githubusercontent.com/omgitsgio/gios_gantry_twist_utility/62a083f4c8356c34b2a8c6e02d26e93530addf10/install.sh | bash
```

> [!NOTE]
> This procedure requires `sudo` access, you will need to have the password for user `mks` at your disposal (if you use certificate based SSH access for instance).

<details>
 
<Summary>Logging example</Summary>

A successful install looks like this:

```log
mks@mkspi:~/printer_data/config/CustomMacros$ cd /home/mks && wget -O - https://raw.githubusercontent.com/omgitsgio/gios_gantry_twist_utility/62a083f4c8356c34b2a8c6e02d26e93530addf10/install.sh | bash
--2026-02-22 11:09:01--  https://raw.githubusercontent.com/omgitsgio/gios_gantry_twist_utility/62a083f4c8356c34b2a8c6e02d26e93530addf10/install.sh
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.109.133, 185.199.111.133, 185.199.108.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.109.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3772 (3.7K) [text/plain]
Saving to: ‘STDOUT’

-                                                    100%[======================================================================================================================>]   3.68K  --.-KB/s    in 0s

2026-02-22 11:09:01 (15.5 MB/s) - written to stdout [3772/3772]


=============================================
- Gantry Twist Utility module install script -
=============================================

[sudo] password for mks:
[PRE-CHECK] Klipper service found! Continuing...

libopenblas-dev is already installed
libatlas-base-dev is already installed
[DOWNLOAD] Downloading Gio's Gantry Twist Utility module repository...
Cloning into 'gantry_twist_utility'...
remote: Enumerating objects: 55, done.
remote: Counting objects: 100% (1/1), done.
remote: Total 55 (delta 0), reused 0 (delta 0), pack-reused 54 (from 1)
Unpacking objects: 100% (55/55), done.
[DOWNLOAD] Download complete!

[SETUP] Installing/Updating Gantry Twist Utility dependencies...
Requirement already satisfied: pip in ./klippy-env/lib/python3.7/site-packages (24.0)
Requirement already satisfied: matplotlib in ./klippy-env/lib/python3.7/site-packages (from -r /home/mks/gantry_twist_utility/requirements.txt (line 1)) (3.5.3)
Requirement already satisfied: numpy in ./klippy-env/lib/python3.7/site-packages (from -r /home/mks/gantry_twist_utility/requirements.txt (line 2)) (1.21.6)
Requirement already satisfied: scipy in ./klippy-env/lib/python3.7/site-packages (from -r /home/mks/gantry_twist_utility/requirements.txt (line 3)) (1.7.3)
Requirement already satisfied: cycler>=0.10 in ./klippy-env/lib/python3.7/site-packages (from matplotlib->-r /home/mks/gantry_twist_utility/requirements.txt (line 1)) (0.11.0)
Requirement already satisfied: fonttools>=4.22.0 in ./klippy-env/lib/python3.7/site-packages (from matplotlib->-r /home/mks/gantry_twist_utility/requirements.txt (line 1)) (4.38.0)
Requirement already satisfied: kiwisolver>=1.0.1 in ./klippy-env/lib/python3.7/site-packages (from matplotlib->-r /home/mks/gantry_twist_utility/requirements.txt (line 1)) (1.4.5)
Requirement already satisfied: packaging>=20.0 in ./klippy-env/lib/python3.7/site-packages (from matplotlib->-r /home/mks/gantry_twist_utility/requirements.txt (line 1)) (24.0)
Requirement already satisfied: pillow>=6.2.0 in ./klippy-env/lib/python3.7/site-packages (from matplotlib->-r /home/mks/gantry_twist_utility/requirements.txt (line 1)) (9.5.0)
Requirement already satisfied: pyparsing>=2.2.1 in ./klippy-env/lib/python3.7/site-packages (from matplotlib->-r /home/mks/gantry_twist_utility/requirements.txt (line 1)) (3.1.4)
Requirement already satisfied: python-dateutil>=2.7 in ./klippy-env/lib/python3.7/site-packages (from matplotlib->-r /home/mks/gantry_twist_utility/requirements.txt (line 1)) (2.9.0.post0)
Requirement already satisfied: typing-extensions in ./klippy-env/lib/python3.7/site-packages (from kiwisolver>=1.0.1->matplotlib->-r /home/mks/gantry_twist_utility/requirements.txt (line 1)) (4.7.1)
Requirement already satisfied: six>=1.5 in ./klippy-env/lib/python3.7/site-packages (from python-dateutil>=2.7->matplotlib->-r /home/mks/gantry_twist_utility/requirements.txt (line 1)) (1.17.0)

[INSTALL] Linking Gantry Twist Utility module to Klipper extras
[POST-INSTALL] Restarting Klipper...
[POST-INSTALL] Restarting Moonraker...
mks@mkspi:~$
```
</details>

 2. Create a new directory and/or a new configuration file

**Directory**
```bash
cd /home/mks/printer_data/config
mkdir -p CustomMacros
```

**File** 
`GiosGantryTwist.cfg` for example:
```cfg
[gantry_twist_utility]

# Operation mode (0 = analysis, 1 = compensation):
# mode: 0

# graphs_folder: ~/printer_data/config/Gantry_twist_analysis

# Mesh boundaries to probe. When in compensation mode, X values will be used for start_x and end_x.
# min_x: 22.0
# max_x: 283.0
# min_y: 22.0
# max_y: 283.0
# calibrate_y: 152.5

# Points per axis (grid_size * grid_size).
# When in compensation mode, this will be the sample size along X-axis.
# grid_size: 10

# Test temps
# bed_temp: 0.0
# hotend_temp: 0.0
```

> [!TIP]
> For more configuration options please refer to [sample_config_complete.cfg](sample_config_complete.cfg).

 3. Include the custom configuration file into `printer.cfg` file with the rest, at the start of the file.
 
```gcode
[include CustomMacros/GiosGantryTwist.cfg] 
```

When you're done editing, be sure to commit the changes to the filesystem

```bash
sync
```

and then power-cycle your printer.

> [!WARNING]
> Power-cycle means to turn your printer off at the power switch, wait 10s, and then turn it back on. It does **NOT** mean just save and restart in the Fluidd webUI.  

### Cleanup

The install script is no longer needed, so you can remove it. Through SSH run:

```
rm -f /home/mks/install.sh
```

## Usage

Just send to your console:

```
GANTRY_TWIST_UTILITY
```

If no settings are declared, it will run with the config above by default (hence in analysis mode).

You can specify some arguments from console which will override the config file:
```
MODE, BED_TEMP, HOTEND_TEMP, GRID_SIZE, MAX_RETRIES, CALIBRATE_Y
```

### Compensation Mode

To automatically calculate and apply axis twist compensation values:

```
GANTRY_TWIST_UTILITY MODE=1
```

This will:
1. Sample along the X-axis at the center Y position as per config
2. Calculate compensation values
3. Update your `[axis_twist_compensation]` configuration
4. Prompt you to run `SAVE_CONFIG` to persist the changes

### Analysis Mode

This mode will probe the bed as per settings/arguments and run 3 dashboards to help visualize the offset data.

Please note that this is all experimental and it's not an exact science, but it has helped me understand that in my case axis twist compensation would have been sufficient to obtain decent results.

### Output Interpretation

The analysis generates three visualization images that help you understand your gantry's twist characteristics:

### 1. `beacon_offset_analysis_3d_meshes_overlay`
Shows overlaid contact and proximity meshes from four different angles, helping you visualize the difference between the nozzle and probe perspectives.

![3D Meshes Overlay](docs/beacon_offset_analysis_3D_meshes_overlay_20251031_234553.png)

### 2. `beacon_offset_analysis_3d_meshes&offset`
Displays:
- Two separate mesh graphs (contact and proximity)
- An overlay view at 45° angle
- A 3D visualization of the deltas (bottom right)

![3D Meshes & Offset](docs/beacon_offset_analysis_3D_meshes&offset_20251031_234553.png)

### 3. `beacon_offset_analysis`
The top-left plot immediately reveals patterns in the extent of the delta values, making it easy to identify bias and twist characteristics.

The X and Y Trend Analysis plots on the right display delta values along their respective axes. The script highlights statistically significant trends and ranges exceeding 50µm. In the example below, note the clear sloping trend on the X-axis and the wide ranges on the Y-axis. While this indicates aggressive gantry twist, the consistent, symmetrical X-axis pattern is ideal for Klipper's X-axis compensation.

The bottom graph in the middle groups the measurements by row or column (depending on the sampling direction) and can help confirm offset patterns.

![Beacon Offset Analysis](docs/beacon_offset_analysis_20251031_234553.png)

## Understanding Gantry Twist Impact on First Layer

Gantry twist directly affects first-layer quality by creating inconsistent Z-offsets across the bed. The following are good examples of some patterns that may occur:

### X-Axis Twist Pattern
When your gantry exhibits twist along the X-axis, you'll see variation in the offset deltas as you move left-to-right across the bed. This is the most common twist pattern on CoreXY printers.

![X-Axis Beacon Offset First Layer](docs/x_beacon_offset_first_layer.png)

### XY Combined Twist Pattern
More complex twist patterns can occur across both X and Y axes, creating diagonal or corner-biased variations that are harder to compensate for mechanically. Generally speaking, you can reduce or remove the effect of one of the axis twists by choosing a beacon/probe mount with zero X or Y offset.

![XY Combined Beacon Offset First Layer](docs/xy_beacon_offset_first_layer.png)

### X-Axis Twist with Compensation
When gantry twist compensation is applied, the first layer becomes more consistent, but the underlying mechanical issue remains.

![X-Axis with Gantry Twist Compensation](docs/x_beacon_offset_gTwist_first_layer.png)

## Uninstall

 1. Remove the `include` line in `printer.cfg` file
 2. Remove the file `CustomMacros/GiosGantryTwist.cfg`
 3. Remove the directory `/home/mks/gantry_twist_utility`

Power-cycle the printer as per step 3 in the installation guide.

## Current Status & Future Plans

### Stability
The script is reliable with default settings or minor variations of them. However, it hasn't been extensively stress-tested with edge cases and misuse, so be careful if you change the defaults.

### Future Improvements
The analysis part is very rudimentary. Suggestions are greatly appreciated.
Potential improvements include:
- Enhanced console output with twist detection alerts
- Automated analysis of twist patterns and severity
- Improved graph layouts and visual storytelling
- Additional visualization types for clearer data interpretation

## Credits
A lot of the problem-solving was possible by taking inspiration from <https://github.com/Frix-x/klippain-shaketune>.

The install script was mostly a readaptation of <https://github.com/qidi-community/ShakeTune-For-Plus4/blob/cd94df7b4b2ac4f6a56c27dfaf9f40dbff463335/install.sh>.

## Support
Please refer to the QIDI community on GitHub: <https://github.com/qidi-community> or join the Discord channel: <https://discord.gg/FtDkxKsX>. Feel free to also contact me on GitHub.


