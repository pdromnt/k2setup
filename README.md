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
## Cleaning up factory gcode (and stopping it from coming back)

1. Open /etc/init.d/klipper in vim
  
2. On line 239, right before the ```sync``` command, comment the rsync command with a hash:

```bash
    # rsync -a $GCODE_PATH/* ${GCODE_FILES_DIR}/
```
3. Delete the files via the printer's screen or Fluidd and reboot printer

## K2Dash

https://github.com/pdromnt/k2dash
