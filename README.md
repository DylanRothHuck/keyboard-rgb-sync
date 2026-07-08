# keyboard-rgb-sync

Sync your laptop keyboard's RGB backlight to your theme accent color.

When you switch themes (e.g. via [Omarchy](https://omarchy.org/) `omarchy theme set`),
this tool reads the theme's accent color, converts it to HSV, and pushes it to the
keyboard via raw HID commands.

## How it works

1. Reads the accent color from the theme (`colors.toml`), falling back to a
   `keyboard.rgb` override file if present
2. Converts the hex color to HSV (hue + saturation)
3. Sends HID write commands to `/dev/hidraw*` to:
   - Set the lighting mode to solid (static single color)
   - Push brightness to maximum
   - Inject the theme's hue and saturation values

## Prerequisites

- Python 3
- A laptop with an RGB keyboard that accepts raw HID commands
  (tested on an ASUS ROG Zephyrus G15 GA503RM)
- `sudo` access to write to `/dev/hidraw*`

## Installation

### 1. Find the correct HID device

```bash
# List HID devices
ls /dev/hidraw*

# Identify your keyboard — run this, then press a key on your keyboard:
cat /dev/hidrawN    # replace N with each number until you see output
```

Once identified, set the `HID_PATH` environment variable when running, or
change the default in the script:

```bash
sudo HID_PATH=/dev/hidraw3 set-keyboard-rgb
```

### 2. Install the script

```bash
# Copy to your local bin directory
cp set-keyboard-rgb ~/.local/bin/
chmod +x ~/.local/bin/set-keyboard-rgb
```

### 3. Test it

```bash
sudo set-keyboard-rgb
```

The script will print the detected accent color and the HSV values being sent.
If you get a "Permission denied" error, run with `sudo`.

## Omarchy integration

If you use [Omarchy](https://omarchy.org/), the hook in `omarchy-hook/` runs
`set-keyboard-rgb` automatically whenever you change themes.

### Install the hook

```bash
cp omarchy-hook/50-keyboard-rgb ~/.config/omarchy/hooks/theme-set.d/
chmod +x ~/.config/omarchy/hooks/theme-set.d/50-keyboard-rgb
```

Now every `omarchy theme set <name>` will sync your keyboard color.

### Override the theme accent

If you want the keyboard to use a different color than the theme accent,
create a `keyboard.rgb` file:

```bash
echo "#ff6600" > ~/.config/omarchy/current/theme/keyboard.rgb
```

The script will use this value instead of the `accent` color from `colors.toml`.

## Manual usage

```bash
# Create a custom keyboard color override
echo "#aabbcc" > ~/.config/omarchy/current/theme/keyboard.rgb

# Run the script
sudo set-keyboard-rgb
```

Or update `keyboard.rgb` in your theme directory and run the script manually
to apply the color without switching themes.

## Troubleshooting

**Permission denied on /dev/hidraw**: The script needs root access to write to
the HID device. Run with `sudo` or set up a udev rule:

```bash
echo 'KERNEL=="hidraw*", SUBSYSTEM=="hidraw", MODE="0660", GROUP="plugdev"' \
  | sudo tee /etc/udev/rules.d/99-keyboard.rules
sudo udevadm control --reload-rules
sudo usermod -aG plugdev $USER
```

Then log out and back in.

**No color change**: Verify the correct `HID_PATH` in the script. Try a different
`/dev/hidraw*` device.
