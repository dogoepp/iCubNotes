**TODO**: check all "TODO" and "FIXME" within this document

# Nancy

## Random remarks

- If the robot is not responding, don't turn it off already (can still press the fault button, though)
    let it run for a while (some minutes)
    maybe a loose ethernet cable is causing very slow communications and errors could show up after a long time

- Old iCubs CPU in head : there was no hard disk. It was remotely mounted using NFs (from server). This means the code was shared with the server computer
    => only the server computer needed to be updated

- yarprobotinterface is a bunch of threads of equal priority but transmission tasks need to be run with high priority. The new version (from June 2017) in icub-main/master fixes this and offers better debug information with respect to it. icub-main/master now handles real-time mode with high priority threads for smoother transmission.

- iCubNancy has "version 1" arms. Joints can be calibrated through similar files as with green/purple iCub (iCubGenova02 and iCubGenova04), in robot-configuration.
    Calibration may done through a calibration matrix found in robots-configuration (for a motor encoder on the leg, a matrix may be found in /hardware/motorControl/x-mc-service.xml, where "x" is the robot part and board number. The parameter named "matrixE2J" is usually set as the identity matrix). For proper explanation on this subject, Valentina Gaggero may be consulted.
    However, for simple fine calibration of joint positions, it is possible to modify the parameter "calibrationDelta" in /calibrators/x-calib.xml, where "x" is the robot part. Description of the procedure is in the wiki (http://wiki.icub.org/wiki/Manual), section "Three. Calibration". 
    After doing this, make sure that the calibration files in the icub-head have the same values as the ones in robot-configuration.


## Notes

Moved source files for PC104 (from icubsrv to icub01 computer)
    (i.e. share code directory between PC104/icub01)
    Paths have changed during this operation. Clean the cmakecache when re-compiling software afterwards.

1. icub server laptop installation instructions 

  - instructions are available on http://wiki.icub.org/wiki/ICub_server_laptop_installation_instructions
  - host file in /etc/hosts -> add line `10.0.0.2 pc104`
  - NFS  server
    1. explanation : icub01 hosts
      - 2 directories : `/exports/code` (source for yarp, icub, etc.) and `/exports/local_yarp` (yarp configuration files, shared between icub01 and PC104)
        > **TODO:** there is a typo on the wiki for these
      - exported using NFS and mounted on PC104
    2. install nfs on icub01
    3. create directories `/exports` and children and set permissions
    4. configure nfs-kernel-server : add lines in `/etc/exports` and restart NFS
  - configure PC104 to mount the remote NFS shares : edit `/etc/fstab` and create mount points (directories)
    > **Caution:** Actually, we should use the facilities of icubrc.d instead of fstab. FIXME
  - on icub01 create a symbolic link to `/exports/code`:
      ```
      sudo ln -s /exports/code /path/to/mount/point
      ```
      (this way, the files are at the same path for icub01 and PC104)
  - clone icub software into `/exports/code` : yarp, icub-main, icub-firmware-shared, icub-firmware-build
  - create a symbolic link to local yarp export path :
      ```
      mkdir -p /home/icub/.local/... FIXME
      sudo ln -s /exports/local-yarp /home/...

  > **Note:** You also want to
  >
  > - install ssh keys for PC104
  > - setup IP forwarding/NAT/IP/DNS/clock sync (NTP)
      ```
2. PC104 installation: this is already done with the Linux image provided by IIT.
3. installation of yarp and icub software on icub server laptop and PC104
    **Note:** for compilation of yarp, the wiki documentation on for the server laptop is not quite up to date. Several additional options (strain, embObjIntertials, embObjMAIS, etc.) need to be set to ON, others need to be OFF (e.g. inertialSensors). Marco Accame should know the details.


## Update firmware

Marco Accame can provide assistance, Valentina Gaggero too.

github.com/robotology/QA/issues/240 : how to update your robot to use 1.8.0; guide to migrate to latest iCub release (applies to CAN and ETH robots)

1. On server computer : pull repositories and compile:
  1. yarp (master)
  2. icub-main (master)

2. On icub-head/PC104,
  1. pull `yarp`, `icub-main`, `icub-firmware-shared`, `icub-firmware-build`, `robots-configuration`
  2. compile yarp
  3. compile yarp-firmware-shared (go to build-pc104, ...)
  4. compile icub-main
  5. install robot-configuration files, using instructions at robotology/robots-configuration
  6. update firmware with FirmwareUpdater (from icub-main/master, replaces ethLoader and CanLoader), following instructions from `icub-firmware-build/firmwareUpdater.readme.quick.txt`
    a. power on motors
    b. start FirmwareUpdater gui on PC104 (through `ssh -X`)
        it should find `firmwareupdate.ini` that should be installed with robots-configuration
    c. select the ETH device and click "Discover" to find available boards. A list of boards appears
    d. select an EMS board; its properties appear on the side (running/idle, etc.)
    e. click **TODO** (the kind of cross) icon to select all boards of the same type
    f. click `Force ETH maintenance`
    g. select all boards in maintenance
    h. click `upload application` and select the `ems.hex` file in `icub-firmware-build` (somewhere in sub-folders)
    i.  click `Discover` again
    j. select each board and check in the properties panel if the firmware version number is correct
    k. select all board of same type with (the kind of cross) icon **TODO**
    l. click `Force ETH application`
        it sends the boards in application state, restarting the application; it is equivalent to power cycle of the motors
    m. Done; quit

3. Use FirmwareUpdater to debug a board which is not responding with `yarprobotinterface`
  1. search board, to see if it finds it
  2. ping board (if not found) -> if it replies -> is `yarprobotinterface` crashed ?
  3. click "restart board" (available if FirmwareUpdater is started with the option "--admin") and check the console for debug info on silence time, timeout, etc.

4. Update loaders and updaters
  > **Caution:** because this process will change board ip address, each board is done one at a time

  This process **must only** be made on **ETH boards**

  Use `FirmwareUpdater --admin`

  1. select ETH + discover (see above)
  2. select a board and press "upload eLoader" and select file from `icub-firmware-build/ETH/EMS/bin/environment/emsLoader.hex` (this step really needs to be done one at at time)
  3. select ETH + discover again; the board shoud appear as EMS `10.0.1.99`
  4. select this board and `force ETH Maintenance`, select again, change IP address to its former value (for instance `10.0.1.1`)
  5. select ETH (unknown), upload application `icub-firmware-build/ETH/EMS/bin/environment/emsApplPROGupdater.hex`

      > **Note:** this is the "special" application allowing to upgrade the eUploader
  6. select ETH devie, `restart ETH boards` and wait ~5s
  7. select ETH device + discover
      you should find eApplPROGupdater (similar name) running
  8. do `Upload eUpdater` with file `icub-firmware-build/ETH/EMS/bin/emsUpdater.hex`
  9. QUICKLY (to stay in eUpdater) select ETH device + discover
  10. select the board being upgraded (for instance `10.0.1.1`) and
      - check in properties that the Updater is on date (e.g. 2016)
          see bootstrap options (FIXME: meaning ? I don't remember. Maybe it is just to check that options are present?)
      - click `jump to eUpdater` to verify that it works : it will change to the eUpdater program
  11. select ETH device + discover + select current device (e.g. `10.0.1.1`)
      - select `set Def Boot eUpdater` (FIXME: will make the board reboot within eUpdater ? I don't think so, but can't remember what it does either)
      - upload appication from `icub-firmware-build/ETH/EMS/bin/application/ems.hex`
      - select the board again and `Force ETH application`
  12. repeat this operation for all boards.


### Update SKIN - for boards with skin

> **Note:** this process could theoretically be done for several boards at a time but there is currently a bug which sometimes prevents the upgrade from being actually done

Boards with skin:
- 2 leftarm
- 4 rightarm
- 10 left leg
- 11 right leg

Use `FirmwareUpdater --admin`
1. select ETH + discover (see above)
2. click `Force ETH maintenance`
2. select a board with skin + discover
3. select all skins of the board (tactile board)
4. click `upload application` and select the `skin.hex` file in `icub-firmware-build` (somewhere in sub-folders)
5. select each board and check in the properties panel if the skin version number is correct
  If the skins are not all the same version, turn off motors. Turn on again and wait (more than 5s).
6. select ETH + `Discover`
7. `Force ETH application`
8. repeat for all boards with skin.

### Other update notes

- MAIS were already at the latest version in June 2017, so they were not updated then.
- STRAIN also need to be updated. This should be done in a similar way as the skin. The concerned boards are 1, 3, 6, 7, 8, 9.
-FOC boards don't need update (at the date of June 2017).

### Force-torque sensors

We manage them through FirmwareUpdater stared with `--admin` option. How it's done:

1. startup `FirmwareUpdater --admin`
2. ETH -> Discover
3. select the desired board among

IP       | side  | part | joints
---------|-------|------|-------
10.0.1.1 | left  | arm  | 0-3
10.0.1.3 | right | arm  | 0-3
10.0.1.6 | left  | leg  | 0-3
10.0.1.7 | left  | leg  | 4-5
10.0.1.8 | right | leg  | 0-3
10.0.1.9 | right | leg  | 4-5

4. click `Force ETH maintenance`
5. select the board and `Discover`
6. select the strain device
7. `Calibrate STRAIN` to launch the strain calibration GUI

This opens the calibration GUI for the force-torque sensors.

What to check:

1. offsets should have a value between 600 and 900
2. calibration matrix should have small values (below FF00 in hex)

The button "automatic offset adjustment" does not seem to work.

> **Note:** The changes are NOT saved until closing the window. At this moment, a prompt asks for confirmation (to save settings).

Playing with offsets will influence the current values of the channels. One can adjust the offsets in order to minimize the channel values. For example, a value above 10 000 is too much. Adjust the slider of each channel until its value is satisfyingly low.

8. Close the GUI and accept to store the changes
9. `Force ETH application`
10. close FirmwareUpdater
11. turn off motors and then back on.

We tuned these values for left leg 0-3, channels 0 and 4.

## Notes for Nancy people

- COPY ALL .local files to git robot-configurations and commit
- will need to upate firmware for board 10.0.1.10 left leg when it arrives from IIT


## How to calibrate IMU
  Put the robot on its pole in home position (i.e. all joints to 0)
  1. Matlab
    1. in a terminal, type `matlab2015b`
    2. in Matlab, go to `codyco-superbuild/main/wBIToolboxControllers/utilities`
    3. launch and run `calibrateIMU.mdl`
    4. look at the IMU scope; the values need to be around
      1. 0 for yellow (x)
      2. 0 for blue (y)
      3. 9.8 for red (z)
  2. with `yarpmotorgui`, move the head joints to bring signals to desired values (said above); keep note of the joint positions (\*)
  3. open file `~/software/yarp/iCuBContrib/robots/iCubNancy01/calibrators/head-calib.xml`
  4. add the offsets to the corresponding joints in `CalibrationDelta`; save and close
  5. restart the robot and check again with step 1.; repeat the steps above as needed since the IMU values should be closer to 0 but they may not yet be perfect
  6. the head uses CAN (not ethernet). The above-mentioned XML file would not be loaded properly. Workaround: put the head in initial position required for the IMU (obtained at (\*) as a **workaround for now**
    This was done with:
    a. **workaround:** `StartupPosition` changed to (\*) in `head-calib.xml`
    b. **workaround:** `homePoseBalancing.ini` is in `.../codyco-superbuild/build/install/share/codyco`. It has been modified according to our measurements (\*): $robot/head-position **was changed** for homePoseYogaPP. Now, joint 0 (original value 0) is set to a new value of -1.5 (measurements (\*) of June 2017) .

## How to run yoga demo

Home position Yoga++; head home position also needs to be adjusted as described above.

1. `yarpmotorgui --from homePoseBalancing.ini`
  -> _menu bar_ global joints commands -> custom positions -> move all parts to YogaPP
2. While the robot is lifted in the air, 
   adjust F/T sensors measurements (adjust offsets) with
  ```
  yarp rpc /wholeBodyDynamics/rpc
  >> calib all 300 # 300 is a timeout in ms
  ```
3. Lower the robot to the ground.
   Perform "makumba" to release eventual stress in the legs.
   Lightly hold the robot by the arms while running the script; it will move a bit
  ```
  twoFeetStandingIdleAndCalib.sh
  ```
  (this script is in the PATH environment variable, its actual path being `/home/icub/software/src/codyco-superbuild/build/install/bin/twoFeetStandingIdleAndCalib.sh`)
  FIXME: update script name to match what it does
4. matlab -> .../codyco-superbuild/main/WBIToobloxControllers/controllers/torqueBalancing
  - torqueBalancingR2015b86.mdl (Simulink file)
  - README (documentation; outdated) FIXME
  - initTorqueBalancing.m (contains a few more lines of doc)
    - check robot name
    - options to turn on/off scopes
      > **Caution:** The scopes should never be opened when the robot is running. Close all scope windows before running the controller.
    - `app/robots/iCubNancy01/gains.m` (controller gains, torque saturation value, ...)
    - `app/robots/iCubNancy01/initStateMachine.m` (gains for yoga state machine, overwrites some variables from `gains.m`)
    - `app/robots/iCubNancy01/initRefGen.m` (more parameters)


## How to run stand up demo

Home position OnChair; head home position also needs to be adjusted as described above.
With WBIToolboxControllers in branch `iCubNancy_standUp_June2017` 
Have the chair ready, and remember to put the plastic covers on the bottom of iCub.

1. `yarpmotorgui --from homePoseBalancing.ini`
  -> _menu bar_ global joints commands -> custom positions -> move all parts to OnChair
2. Lower the robot on the chair. Add some panels between the chair and iCub, to make sure that the feet do not touch the ground.
3. Adjust F/T sensors measurements (adjust offsets) with
  ```
  yarp rpc /wholeBodyDynamics/rpc
  >> calibStandingOnTwoLinks l_upper_leg_contact r_upper_leg_contact 100 # 100 is a timeout in ms
  ```
3. Remove the panels from under iCub. Place the robot so that the two feet are flat on the ground.
4. Perform "makumba" to release eventual stress in the legs.
   Lightly hold the robot by the arms while running the script; it will move a bit
  ```
  twoFeetStandingIdleAndCalib.sh
  ```
  (this script is in the PATH environment variable, its actual path being `/home/icub/software/src/codyco-superbuild/build/install/bin/twoFeetStandingIdleAndCalib.sh`)
  FIXME: update script name to match what it does
4. matlab -> .../codyco-superbuild/main/WBIToobloxControllers/controllers/torqueBalancing
  - torqueBalancingR2016.mdl (Simulink file)
  - ICUB_STANDUP_README (documentation)
  - initTorqueBalancing.m (contains a few more lines of doc)
    - check robot name
    - check SM.SM_TYPE = 'STANDUP'
    - options to turn on/off scopes
      > **Caution:** The scopes should never be opened when the robot is running. Close all scope windows before running the controller.
    - `app/robots/iCubNancy01/gains.m` (controller gains, torque saturation value, ...)
    - `app/robots/iCubNancy01/initStateMachine.m` (gains for yoga state machine, overwrites some variables from `gains.m`)
    - `app/robots/iCubNancy01/initRefGen.m` (more parameters)


FIXME Some hardware joint limits were changed in PID file (from robot-configurations). Double check this.

## Changes in robot-configuration motor control (PID gains from yarpmotorgui -> new change)

| robot part | joint           | kp              | Kbemf  | ktau   |
|------------|-----------------|-----------------|--------|--------|
| torso      | yaw             | 450             | 0.0008 | 200    |
| torso      | roll            | 400             | 0.0015 | 200    |
| torso      | pitch           | 400             | 0.0015 | 200    |
| l_arm      | shoulder, elbow |                 | 0      |        |
| r_arm      | shoulder, elbow |                 | 0      |        |
| l_leg      | knee            | switch sign     |        | -100   |
| r_leg      | r_ankle_pitch   | -300 (was -200) |        | &nbsp; |


## Sidenotes

> **Note:** issue to open: permettre de saisir au clavier des valeurs désirées pour les articulations dans yarpMotorGui

> **Note:** Issue to open : robot-config does not install firmwareupdate.ini (check CMMakeLists file)

> **Note:** Ajouter une issue pour que les informations et les instructions de mise à jour ne soient pas enregistrées dans des issues github à cause de leur nature périssable.

> **Note:** `yarpscope --remote /icub/left_arm/analog:o`

> **Note:** Issue to open? : sometimes after starting the robot, in iCubGui, the force on the right leg is not visible. `yarp read` on the right leg gives `NaN` and there are no errors on the logger. Is this something to do with the F/T sensors calibration?

FIXME: "BUG sm.com.threshold TWICE in initStateMachine.m"