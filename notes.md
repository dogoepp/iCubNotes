**TODO**: check all "TODO" and "FIXME" within this document

# Nancy

## Random recommendations

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



> **Note:** Issue to open : robot-config does not install firmwareupdate.ini (check CMMakeLists file)


> **Note:** Ajouter une issue pour que les informations et les instructions de mise à jour ne soient pas enregistrées dans des issues github à cause de leur nature périssable.
