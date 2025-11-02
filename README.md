# Customizations for my QIDI Plus 4 3d printer

⚠️ **Do not follow these steps if on firmware earlier than v1.7.1 !**

⚠️ **The following configuration changes can cause harm to you and/or those around you. Please understand and only do this if you are confident in what you are doing. I accept no liability for damage or harm to you, others or your property. If you are in doubt, do not perform this mod.**

## Configuration changes

### printer.cfg

```
[z_tilt]
speed: 300

[extruder]
microsteps: 32

[tmc2209 extruder]
interpolate: False
run_current: 0.8

[tmc2240 stepper_y]
interpolate:False
stealthchop_threshold: 0
run_current: 1.15

[tmc2240 stepper_x]
interpolate:False
stealthchop_threshold: 0
run_current: 1.15

[tmc2209 stepper_z]
interpolate: False
stealthchop_threshold: 0
run_current: 1.15

[tmc2209 stepper_z1]
interpolate: False
stealthchop_threshold: 0
run_current: 1.15

[heater_generic chamber]
max_power:0.7

[smart_effector]
speed:2.5
lift_speed:20

[bed_mesh]
speed:300
horizontal_move_z:4
```

##### Notes

1. **interpolate** is set to False for all steppers.
2. **stealthchop_threshold** is disabled for all motors, for better torque and precision, with the expense of higher noise on z-axis motors. I wish QIDI could switch to TMC2240 for all X,Y,Z motors. 
3. **max_power** for the chamber heater is set to 0.7.
4. **smart_effect.speed** in the section is changed down to **2.5** from 5, to reduce the probe speed, to get more precise z probes.
5. **smart_effect.lift_speed=20** , **bed_mesh.speed=300**, and **bed_mesh.horizontal_move_z=5** for faster auto bed leveling.
6. Do remember to re-run the viberation calibration after the above changes.

```
[screws_tilt_adjust]
screw1:0,20
screw1_name: Front left
screw2: 260,20
screw2_name: Front right
screw3: 260,280
screw3_name: Back right
screw4: 0,280
screw4_name: Back left
screw_thread: CW-M4

[gcode_macro SCREWS_TILT_CALCULATE]
rename_existing: _SCREWS_TILT_CALCULATE_BASE
gcode:
    { action_respond_info("starting screw rotation calculation...") }
    M141 S0 # disable chamber heater (see https://github.com/qidi-community/Plus4-Wiki/tree/main/content/chamber-heater-issue)
    M4031
    G28
    _SCREWS_TILT_CALCULATE_BASE
    
```
#### Notes
The above config is copied from [Enabling SCREWS_TILT_CALCULATE](https://github.com/qidi-community/Plus4-Wiki/tree/main/content/Screws-Tilt-Adjust).

### box?.cfg

```
[box_heater_fan heater_fan_a_box1]
heater_temp: 40

[box_heater_fan heater_fan_b_box1]
heater_temp: 40

```

##### Notes

1. **heater_temp** is changed to 40 degrees up from 35. The default setting will make QIDI box heater fan running all the time.

## Code changes

The following code in klipper/klippy/extras/heaters.py

```
        #    temp, read_time, temp_diff, temp_deriv, temp_err, temp_integ, co)
        bounded_co = max(0., min(self.heater_max_power, co))
        if self.heater.name == "chamber" and heater_bed.heater_bed_state != 2 and heater_bed.is_heater_bed == 1:
            self.heater.set_pwm(read_time, 0.)
        else:
            self.heater.set_pwm(read_time, bounded_co)

```

I changed the above codes to be :

```
        #    temp, read_time, temp_diff, temp_deriv, temp_err, temp_integ, co)
        bounded_co = max(0., min(self.heater_max_power, co))
        if self.heater.name == "chamber" and heater_bed.heater_bed_state != 2 and heater_bed.is_heater_bed == 1:
            self.heater.set_pwm(read_time, bounded_co/2.0)
        else:
            self.heater.set_pwm(read_time, bounded_co)
```

The change enables the chamber heater to work together with the bed heater to speed up the chamber warming process.

## Pre-warm the bed for high temp filaments like ABS, etc.

[Customized PRINT_START marco in gcode_marco.cfg](https://github.com/QIDITECH/QIDI_PLUS4/pull/100)

## Other Hardware Mods
1. [Rear chamber cover](https://www.printables.com/model/1040774-qidi-plus-4-rear-chamber-cover)
2. [Filament waste container](https://www.printables.com/model/1023520-qidi-plus-4-filament-waste-container)
3. [Mainboard fan mod](https://www.printables.com/model/1146502-qidi-plus-4-hex-mainboard-cover-for-80mm-fan-with) with [Fan dust filter] (https://makerworld.com/en/models/967259-80-92-120-140mm-fan-dust-filter#profileId-938179)
4. [Filtration Mod](https://www.printables.com/model/1022271-qidi-plus-4-filtration-system)
5. [Wire ramp](https://www.printables.com/model/1094939-qidi-plus-4-wire-ramp)
6. [Top glass stand](https://www.printables.com/model/1102150-qidi-plus-4-top-glass-stand)
7. [Hotend reinforcement, CNCed version](https://makerworld.com.cn/zh/models/1021954-plus4-re-duan-jia-qiang)
8. [Bed Screw Adjustment Tool for QIDI Plus 4](https://www.printables.com/model/1280126-bed-screw-adjustment-tool-for-qidi-plus-4)
9. [Nevermore Micro V6](https://github.com/nevermore3d/Nevermore_Micro/tree/master/V6) sharing 24V power with the LED lights on the mainboard.
10. Comfast USB WIFI6 AX600 Adapter
