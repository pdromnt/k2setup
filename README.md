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

Step 1 — pick a name and number. We're picking S99 so it runs after most of everything already setup from factory on the printer.
  
Step 2 — the script. Save this to /etc/init.d/S99gcode-cleanup on the printer:

```bash
#!/bin/sh /etc/rc.common

# Remove the three Creality ROM-overlay demo gcodes. Creality's firmware
# re-copies these from ROM to /mnt/UDISK/printer_data/gcodes on every
# boot, which clutters the file list shown in the k2-dash dashboard.
# Re-runnable: rm -f is a no-op on missing files.

START=90

PRINTER_GCODES=/mnt/UDISK/printer_data/gcodes

start() {
    rm -fv \
        $PRINTER_GCODES/3DBench_PLA_21m47s.gcode \
        $PRINTER_GCODES/3DBenchy_PLA_34m1s.gcode \
        $PRINTER_GCODES/Crealitylogo_PLA_32m41s.gcode \
        >> /tmp/k2dash-init.log 2>&1
}

stop() {
    # Nothing to undo on shutdown — the next boot will re-clean.
    return 0
}
```

Step 3 — install it. Paste these into SSH:
1. Write the file (paste the script above into a heredoc).
```
cat > /etc/init.d/S99gcode-cleanup <<'EOF'
[... paste the script ...]
EOF
```

2. Make it executable.  
```chmod +x /etc/init.d/S99gcode-cleanup```

3. Register it with the runlevel system (BusyBox-style on K2 Plus).  
```ln -s /etc/init.d/S99gcode-cleanup /etc/rc.d/S99gcode-cleanup```

4. Run it now to clean up right now.  
```/etc/init.d/S99gcode-cleanup start```

5. Confirm the files are gone.  
```ls -la /mnt/UDISK/printer_data/gcodes/ | grep -E '3DBench|3DBenchy|Crealitylogo' || echo "all three gone"```

## K2Dash

https://github.com/pdromnt/k2dash
