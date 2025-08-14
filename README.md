# Customizations for my QIDI Plus 4 3d printer

⚠️ **Do not follow these steps if on firmware earlier than v1.7.1 !**

⚠️ **The following configuration changes can cause harm to you and/or those around you. Please understand and only do this if you are confident in what you are doing. I accept no liability for damage or harm to you, others or your property. If you are in doubt, do not perform this mod.**

## Configuration changes

### printer.cfg

```
[stepper_z]
microsteps: 64

[stepper_z1]
microsteps: 64

[tmc2240 stepper_y]
interpolate:False

[tmc2240 stepper_x]
interpolate:False

[tmc2209 stepper_z]
interpolate: False
stealthchop_threshold: 0

[tmc2209 stepper_z1]
interpolate: False
stealthchop_threshold: 0

[heater_generic chamber]
max_power:0.6
heat_with_heater_bed_tem_add:15

[smart_effector]
speed:2.5

[bed_mesh]
probe_count:11,11


```

##### Notes

1. **interpolate** is set to False for all steppers.
2. **stealthchop_threshold** is disabled for z motors, for better torques, with the expense of higher noise.
3. **max_power** for the chamber heater is set to 0.6, cause I am in China, the electricity is 220 voltage.
4. **heat_with_heater_bed_tem_add** is set to **15**, down from 25. The 1.7.1 version firmware introduced this new config. The chamber heater will not be enabled unless the bed reaches the target chamber temperature + heat_with_heater_bed_tem_add. This makes the chamber warming process too slow. I just lowered down this value, and enabled the chamber heater with half of its max_power, to speed up the chamber warming process.
5. **speed** is changed down to **2.5** from 5, to reduce the probe speed, to get more precise z probes.
6. **probe_count** is changed from 9,9 to 11,11, to get better bed mesh, with the expense of longer probe process.

```
[controller_fan board_fan]
pin:U_1:PB2

[temperature_sensor mainboard_stm32]
sensor_type: temperature_mcu
sensor_mcu: U_1

[temperature_sensor mainboard_rockchip]
sensor_type: temperature_host

[temperature_fan adaptive_board_fan]
pin:U_1:PC4
max_power: 1.0
shutdown_speed: 1.0
cycle_time: 0.3
sensor_type: temperature_combined
sensor_list: temperature_sensor mainboard_stm32, temperature_sensor mainboard_rockchip
combination_method: max
maximum_deviation: 999.9
control: pid
pid_deriv_time: 5.0
pid_Kp: 5
pid_Ki: 2
pid_Kd: 5
target_temp: 50
min_speed: 0
max_speed: 1.0
min_temp: 0
max_temp: 100


#[controller_fan board_fan]
#pin:U_1:PC4
#max_power:1.0
#shutdown_speed:1.0
#cycle_time:0.01
#fan_speed: 1.0
#heater:chamber
#stepper:stepper_x,stepper_y
```

##### Notes
1. The above configuration is copied from https://github.com/qidi-community/config-xplus4/blob/main/board-fan-cooling.cfg

### box?.cfg

```
[box_heater_fan heater_fan_a_box1]
heater_temp: 40

[box_heater_fan heater_fan_b_box1]
heater_temp: 40

```

##### Notes

1. heater_temp is changed to 40 degrees up from 35. The default setting will make QIDI box heater fan running all the time.

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
