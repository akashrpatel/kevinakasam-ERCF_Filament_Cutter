# Filament Cutter for the ERCF

This is a simple filament cutter for the awesome ERCF so tip tuning is less important.

It's made out of a Servo and a scalpell/exacto knife blade. The cutter get's mouted to the selector chart just behind the encoder - the cutter is basically an arm that swings to the left and right. Since the ECAS sits now on the Arm the filament path can be opened by swining the arm to the side. Then the ERCF can feed some filament mid air (*the filament tip*) and when the arm swings back to close the filament path the tip gets cut off by the knife on the arm. The cut off tip just falls off to the ground and the filament can be loaded with a clean tip the next time.

#### Why?
I never thought about a filament cutter untill I saw the [Filametrix](https://github.com/sorted01/Filametrix) design / got it recommended. Since I'm not using the ERCF with a Stealthburner (for now at least) I would have to make a cutter on my own. At the same time u/[BioKeks](https://github.com/BioCookieYT) requested a cutter for my Ender 3 fan duct ([Frankenstein 2.0](https://github.com/kevinakasam/FrankEnstein-Duct/tree/main/Frank2.0_Beta)) and that way I though we need something universial. After some days of tinkering this was the result - first with the MG90s servo, but that was so weak it was far aways from cutting it. So long speech short, thanks to [sorted01](https://github.com/sorted01) for the idea with the cutter!

#### Credits

Thanks to u/[BioKeks](https://github.com/BioCookieYT) for tinkering with me! Luckily we both got the ERCF roughly at the same time :D

Thanks to u/[sorted01](https://github.com/sorted01) for the idea with the cutter!

Thanks to u/[moggieuk](https://github.com/moggieuk) for the [awesome firmware](https://github.com/moggieuk/Happy-Hare/tree/23604442613be6bbbc8c70f57ff55fe65b1268e4) and the quick help on the Voron Discord!

And of course thanks to u/[EtteGit](https://github.com/EtteGit) for the awesome [EnragedRabbitProject](https://github.com/EtteGit/EnragedRabbitProject)!

![](Images/Render.png)

 <img src="Images/Cutter1.jpg" width=49% >  <img src="Images/Cutter2.jpg" width=49% >

 #### Cutter Action:
 
<a href="http://www.youtube.com/watch?feature=player_embedded&v=-_qIqNOiTIw
" target="_blank"><img src="Images/yt.png" 
alt="Cutter Preview" width="32%" border="10" /></a>
 

### BOM

| Qty | Part | Description 
| --- | --- | --- |
| 1 | [Servo MR996](https://amzn.eu/d/gEQACnZ) | That's just the servo I use, nothing special. But the MG90S are defineetly too weak. 60° servos are fine, travel is limited by design anyways. |
| 1 | [Exacto knife blade No. 11](https://amzn.eu/d/0ZmUXIy) | That's just an example so you know the shape/type. I didn't test these! A higher quality blade probably last longer. |
| 1 | 5V buck converter | The Servo can take quite a lot of current, enough to shut down my ERCF board. If you have the same problem try a buck converter wired to the PSU |
| ... | ... | I will add more later but it's just some screws, inserts and washers |

### To-Do
- Move blade a little so the screw can't collide with the PCB
- Mirror the cutter so the cutting force doesn't act against the screw thread on the servo arm (can't unscrew itself)
- Fixed an overhang issue with the `Servo_Mount.stl`

### Klipper Stuff

I'm using the awesome [Happy Hare](https://github.com/moggieuk/Happy-Hare) firmware for the ERCF. I'm sure this also works with the stock macros, but I haven't tried that. In addition, the Happy Hare `[mmu_servo]` works a lot better than the stock `[servo]` from Klipper. I added a file named `mmu_ercf_cutter.cfg` in the ` /config/mmu/optional` to implement it with a `[include mmu/optional/mmu_ercf_cutter.cfg]`. 
Note: The Macros aren't done yet and aren't working as expected.

```
[mmu_servo cut_servo]         #`mmu_servo` only for the Happy Hare firmware, otherwise only `servo`
pin: mmu:PA7                  #Extra Pin on the ERCF easy Board
maximum_servo_angle: 180      #Set this to 60 for a 60° Servo
minimum_pulse_width: 0.0005   #Adapt these for your servo
maximum_pulse_width: 0.0025   #Adapt these for your servo

[gcode_macro _CUT_VAR]
description: Empty macro to store the variables

### Servo ###
variable_servo_closed_angle: 70          #Adapt these for your setup
variable_servo_open_angle: 10            #Adapt these for your setup
variable_servo_blocked_angle: 40         #I have the idea to home the filament against the cutter by blocking the filament path with 1/2 of the servo rotation. Could make cuts more precise
variable_servo_idle_time: 1000           #Time to let the servo reach it's position, in milliseconds (1 second = 1000 milliseconds)

### Feed lengths
variable_cut_length: 10                  #How much you want to cut off in mm
gcode:


[gcode_macro CUTTER_CLOSE]
description: Set Servo in the close position
gcode:
    {% set cutvar = printer["gcode_macro _CUT_VAR"] %}
    SET_SERVO SERVO=cut_servo ANGLE={cutvar.servo_closed_angle}
    SET_SERVO SERVO=cut_servo WIDTH=0.0
    G4 P{cutvar.servo_idle_time}
    M118 Cutter closed
    M400
    G4 P0

[gcode_macro CUTTER_OPEN]
description: Set Servo in the open position
gcode:
    {% set cutvar = printer["gcode_macro _CUT_VAR"] %}
    SET_SERVO SERVO=cut_servo ANGLE={cutvar.servo_open_angle}
    SET_SERVO SERVO=cut_servo WIDTH=0.0
    G4 P{cutvar.servo_idle_time}
    M118 Cutter open
    M400
    G4 P0

[gcode_macro CUTTER_BLOCK]
description: Set Servo in the blocked position
gcode:
    {% set cutvar = printer["gcode_macro _CUT_VAR"] %}
    SET_SERVO SERVO=cut_servo ANGLE={cutvar.servo_blocked_angle}
    SET_SERVO SERVO=cut_servo WIDTH=0.0
    G4 P{cutvar.servo_idle_time}
    M118 Cutter blocked
    M400
    G4 P0

[gcode_macro CUTTER_Action]
description: Set Servo in the close position. Doesn't work properly yet.
gcode:
    {% set cutvar = printer["gcode_macro _CUT_VAR"] %}

    MMU_SERVO POS=down
    CUTTER_OPEN
    MMU_TEST_MOVE MOVE={44 + cutvar.cut_length} #Rough estimation of the parking position to the outout of the encoder + the cut length
    CUTTER_CLOSE
    CUTTER_OPEN
    CUTTER_CLOSE
    CUTTER_OPEN
    MMU_TEST_MOVE MOVE=-1
    CUTTER_CLOSE
    MMU_EJECT
    #MMU_SERVO POS=down
    M400
    G4 P0
```

### Changelog

30/09/2023: Blinky Support
- Adaption of the Blinky chart for the cutter - made by u/[BioKeks](https://github.com/BioCookieYT)
