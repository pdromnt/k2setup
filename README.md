# What is this?
This is my custom setup for my Creality K2 Plus. Some people would probably benefit from using some of those things (like the DXC2 config that allows you to use Creality's cutters) or my own project, the K2Dash.

Enjoy. Please don't open PRs or issues.

## DXC2

### box.cfg

````
Tn_retrude: -10 -> Tn_retrude: -25  
switch_pin:!nozzle_mcu:PB9 -> switch_pin:nozzle_mcu:PB9 # Allows you to use Creality's own cutters
````

Optional, this allows you to use Creality's cutters instead of the DXC2 cutter if you want without needing to transplant the blade of inverting the magnet:

````
switch_pin:!nozzle_mcu:PB9 -> switch_pin:nozzle_mcu:PB9
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
