## DXC2 Setup

### box.cfg

Tn_retrude: -10 -> Tn_retrude: -25  
switch_pin:!nozzle_mcu:PB9 -> switch_pin:nozzle_mcu:PB9

### gcode_macro.cfg -> [gcode_macro QUIT_MATERIAL_RETRUDE_MATERIAL]

G0 E-10 F360 -> G0 E-40 F360

## Auto Cleanup

Step 1 — pick a name and number. Order matters: lower numbers run first. Klipper is S55klipper and the moonraker-style printer service is probably in the 50s-60s range. We need to run before klipper starts (so the dashboard is clean by the time it shows the file list) but after the filesystem is mounted writable. S50klipper-cleanup is a safe slot — runs at the same priority as klipper, alphabetically first.
If you want to be extra-safe and run before klipper, S49klipper-cleanup (Klipper is S55, so S49 runs first). I'd go with that.
Step 2 — the script. Save this to /etc/init.d/S49klipper-cleanup on the printer (the file name is /etc/init.d/S49klipper-cleanup):
#!/bin/sh

# Remove the three Creality ROM-overlay demo gcodes. Creality's firmware
# re-copies these from ROM to /mnt/UDISK/printer_data/gcodes on every
# boot, which clutters the file list shown in the k2-dash dashboard.
# Re-runnable: rm -f is a no-op on missing files.

PRINTER_GCODES=/mnt/UDISK/printer_data/gcodes

DEMO_GCODES="
  $PRINTER_GCODES/3DBench_PLA_21m47s.gcode
  $PRINTER_GCODES/3DBenchy_PLA_34m1s.gcode
  $PRINTER_GCODES/Crealitylogo_PLA_32m41s.gcode
"

case "$1" in
  start)
    # Log what we removed (or would have) to a tmpfs file so it
    # survives in dmesg/journal but doesn't fill /var/log.
    rm -fv $DEMO_GCODES >> /tmp/k2dash-init.log 2>&1
    ;;
  stop|restart|reload)
    # Nothing to undo on shutdown — the next boot will re-clean.
    ;;
  *)
    echo "Usage: $0 {start|stop|restart|reload}" >&2
    exit 1
    ;;
esac

exit 0
Step 3 — install it. Paste these into SSH:
# 1. Write the file (paste the script above into a heredoc).
cat > /etc/init.d/S49klipper-cleanup <<'EOF'
[... paste the script ...]
EOF

# 2. Make it executable.
chmod +x /etc/init.d/S49klipper-cleanup

# 3. Register it with the runlevel system (BusyBox-style on K2 Plus).
update-rc.d S49klipper-cleanup defaults 2>/dev/null || /etc/init.d/rcS 2>/dev/null
# If neither works on the Creality rootfs, just call it from
# /etc/init.d/rcS or from S55klipper's start() case.

# 4. Run it now to clean up right now.
/etc/init.d/S49klipper-cleanup start

# 5. Confirm the files are gone.
ls -la /mnt/UDISK/printer_data/gcodes/ | grep -E '3DBench|3DBenchy|Crealitylogo' || echo "all three gone"

## K2Dash

https://github.com/pdromnt/k2dash
