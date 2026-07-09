## DXC2

### box.cfg

````
Tn_retrude: -10 -> Tn_retrude: -25  
switch_pin:!nozzle_mcu:PB9 -> switch_pin:nozzle_mcu:PB9 # Allows you to use Creality's own cutters
````
### gcode_macro.cfg -> [gcode_macro QUIT_MATERIAL_RETRUDE_MATERIAL]
````
G0 E-10 F360 -> G0 E-40 F360
````
## Auto Cleanup

Step 1 — pick a name and number. We're picking S90 so it runs after most of everything already setup from factory on the printer.
  
Step 2 — the script. Save this to /etc/init.d/S90klipper-cleanup on the printer (the file name is /etc/init.d/S90klipper-cleanup):

```bash
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
```

Step 3 — install it. Paste these into SSH:
1. Write the file (paste the script above into a heredoc).
```
cat > /etc/init.d/S90klipper-cleanup <<'EOF'
[... paste the script ...]
EOF
```

2. Make it executable.  
```chmod +x /etc/init.d/S90klipper-cleanup```

3. Register it with the runlevel system (BusyBox-style on K2 Plus).  
```ln -s /etc/init.d/S90klipper-cleanup /etc/rc.d/S90klipper-cleanup```

4. Run it now to clean up right now.  
```/etc/init.d/S90klipper-cleanup start```

5. Confirm the files are gone.  
```ls -la /mnt/UDISK/printer_data/gcodes/ | grep -E '3DBench|3DBenchy|Crealitylogo' || echo "all three gone"```

## K2Dash

https://github.com/pdromnt/k2dash
