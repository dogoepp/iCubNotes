**TODO**: check all "TODO" and "FIXME" within this document

# Nancy

## Random recommendations

- If the robot is not responding, don't turn it off already (can still press the fault button, though)
    let it run for a while (some minutes)
    maybe a loose ETH cable is causing very slow communications and errors could show up after a long time

- Old iCubs CPU in head : there was no hard disk. It was remotely mounted using NFs (from server). This means the code was shared with the server computer
    => only the server computer needed to be updated

- yarprobotinterface is a bunch of theads of equal priority but transmission tasks need to be run with high priority. The new version in icub-main/master tixes this and offers better debug information with respect to it

- icub-main/master now handles RT (FIXME: real-time ?) mode with high priority threads for smoother transmission

- iCub Nancy has "version 1" arms. Joints can be calubrated through similar files as with freen/purple iCub, in robot-configuration.
    Calibration is done through matrixe23 (FIXME: not sure about this word) (e.g.) motor encoder -> joint by setting the first 4 columns = 1/4 ... and then ask Julien/Valentina (or changing delta if lazy)
    FIXME : I don't understand this note. Could you help, Marie ?

## On issue documentation

yarp -> strain, embObjIntertials, embObjMAIS... ON
inertialSensors OFF
FIXME: is this something important for us ? meaning ?

## Update firmware

Marco Accame can provide assistance, Valentina Gaggero too.

github.com/robotology/QA/issues/240 : how to update your robot to use 1.8.0; guide to migrate to latest iCub release (applies to CAN and ETH robots)

1. On server computer : pull repositories and copile:
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
        e. click **TODO** icon to select all boards of the same type
        f. click "Force ETH maintenance"
        g. select all boards in maintenance
        h. click "upload application" and select the `ems.hex` file in `icub-firmware-build` (somewhere in sub-folders)
        i. do a discovery again
        j. select each board and check in the properties panel if the firmware version number is correct
        k. select all board of same type with icon **TODO**
        l. click "Force ETH application"
            it sends the boards in application state, restarting the application; it is equivalent to power cycle of the motors
        m. Done; quit
3. Use FirmwareUpdater to debugg a board which is not responding with `yarprobotinterface`
    1. search board, to see if it finds it
    2. ping board (if not found) -> if it repies -> is `yarprobotinterface` crashed ?
    3. click "restart board" (available if FirmwareUpdater is started with the option "--admin") and check the console for debug info on silence time, timeout, etc.

4. Update loarders and updaters
    > **Caution:** because this process will change board ip address, each board is done one at a time

    This process **must only** be made on **ETH boards**

    Use `FirmwareUpdater --main` (is it not "admin" ?)

    1. select ETH + discover (see above)
    2. select a board and press "upload eLoader" and select file from `icub-firmware-build/ETH/EMS/bin/environment/emsLoader.hex` (this step really needs to be done one at at time)
    3. select ETH + disvover again; the board shoud appear as EMS `10.0.1.99`
    4. select this board and "force ETH Maintenance", select again, change IP address to its former value (for instance `10.0.1.1`)
    5. select ETH (unknown), upload application `icub-firmware-build/ETH/EMS/bin/environment/emsApplPROGupdater.hex`

        > **Note:** this is the "special" application allowing to upgrade the eUploader
    6. select ETH devie, restart ETH boards and wait ~5s
    7. select ETH device + discover
        you should find eApplPROGupdater (similar name) running
    8. do "Upload eUpdater" with file `icub-firmware-build/ETH/EMS/bin/emsUpdater.hex`
    9. QUICKLY (to stay in eUpdater) select ETH device + discover
    10. select the board being upgraded (for instance `10.0.1.1`) and
        - check in properties that the Updater is on date (e.g. 2016)
            see bootstrap options (FIXME: meaning ?)
        - click "jump to eUpdater" to verify that it works : it will change to the eUpdater program
    11. select ETH device + discover + select current device (e.g. `10.0.1.1`)
        - select "set Def Boot eUpdater" (FIXME: will make the board reboot within eUpdater ?)
        - upload appication from `icub-firmware-build/ETH/EMS/bin/application/ems.hex` (this step really needs to be done one at at time)
        - select the board again and "Force ETH application" (FIXME: restart ?)
    12. repeat this operation for all boards.

    > **Note:** this process could theoretically be done for several boards at a time but there is currently a bug which sometimes prevents the upgrade from being actually done

### Boards with skin

FIXME: left hand side of the page not scanned (board numbers)

Boards:

- ***, 2, 4
- 2 leftarm
- 4 rightarm
- 11 right leg
- 10 left leg
- *** -> already latest version

Procedure:

1. force ETH maintenance
2. select 4 discover (FIXME: what is this "4" here ?)
3. select all skin boards (tactile board)
    > **Crucial:** if not all at same version, turn off motors. Turn on again and wait (more than 5s).
    > **Caution:** doing software updates in parallel might cause *** to happen; better do it one by one or selecting boards with non consecutive IPs (might also fail)
4. ETH + Discover
5. upload application `icub-firmware-build/skin.hex`

Repeat for all boards to be updated

## Notes

Move sourc efiles for PC104 (from icubsrv to icub01 computer)
    (i.e. share code directory between PC104/icub01)

(cmakecache <- clean for paths wichi have changed) FIXME: what does it mean ?

1. icub server laptop installation isntructions
    - host file in /etc/hosts -> add line `10.0.0.2 pc104`
    - NFS  server
        1. explanation : icub01 hosts
            - 2 directories : `/exports/code` (source for yarp, icub, etc.) and `/exports/local_yarp` (yarp configuration files, shared between icub01 and PC104)
                > **FIXME:** there is a typo on the wiki for these
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
        ```
2. PC104 installation instructions: should be done already. FIXME: what are those ?
3. installation of software FIXME: ???

> **Note:** You also want to
>
> - install ssh keys for PC104
> - setup IP forwarding/NAT/IP/DNS/clock sync (NTP)

## Force-torque sensors

We manage them through FirmwareUpdater stared with `--admin` option. How it's done:

1. startup `FirmwareUpdater --admin`
2. ETH -> Discover
3. select the desired board among
    
    IP       | side  | part | joints
    -------- | ----- | ---- | ------
    10.0.1.1 | left  | arm  | 0-3
    10.0.1.3 | right | arm  | 0-3
    10.0.1.6 | left  | leg  | 0-3
    10.0.1.7 | left  | leg  | 4-5
    10.0.1.8 | right | leg  | 0-3
    10.0.1.9 | right | leg  | 4-5

4. click "Force ETH maintenance"
5. select the board and "Discover"
6. select the strain device
7. "Calibrate STRAIN" to launch the strain calibration GUI

This opens the calibration GUI for the force-torque sensors.

What to check:

1. offstes should have a value between 600 and 900
2. calibration matrix should have small values (bellow FFXX in hex)
    FIXME: what is the XX for in hex ?

The button "automatic offset adjustment" does not seem to work.

> **Note:** The changes are NOT saved until closing the window. At htis moment, a prompt asks for confirmation (to save settings)

Playing with offsets will influence the current values of the channels. One can adjust the offsets in order to minimize the channel values. For example, a value above 10 000 is too much. Adjust the slider of each channel until its value is satisfyingly low.

8. Close the GUI and accept to store the changes
9. force eth application
10. close firmware updater
11. turn off motors and then back on.

We tuned these values for left leg 0-3, channels 0 and 4.

## Notes for Nancy people

- COPY ALL .local files to git robot-configurations and commit
- 




> **Note:** Issue to open : robot-config does not install firmwareupdate.ini (check CMMakeLists file)


> **Note:** Ajouter une issue pour que les informations et les instructions de mise à jour ne soient pas enregistrées dans des issues github à cause de leur nature périssable.

> **Note:** `yarpscope --remote /icub/left_arm/analog:o`