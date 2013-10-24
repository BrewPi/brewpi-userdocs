Changelog
=========

October 22nd, 2013
------------------
It's been a long time since a master release, but in this release we made some major steps to make releasing easier in the future.

The biggest changes in this release are:

* Added a native interface for beer profiles. No more google spreadsheets!
  Huge thank you to Brian Schwinn (bschwinn) for his hard work on this feature.
* Added an easy menu to start a new brew, start/stop/pause data logging
* Added an install and update script! Just run the script and afterwards go straight to flashing your Arduino and setting up devices.
  Huge thank you to Geo van O. (Freeder)  for his hard work on these scripts.
* Tweaked the temperature control algorithm to reduce overshoot.

Instructions for installing/updating BrewPi can be found in the documentation.
and the scripts are part of the new `brewpi-tools repository on GitHub <https://github.com/BrewPi/brewpi-tools>`_.

Detailed changes per repository are displayed below.

Web interface (brewpi-www) | 0.2 -> 0.3
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* New interface to create/edit/save/load profiles
* Added dialog to start/stop/pause data logging and to start a new brew
* Split config files in default config in source control and user config outside of source control
* Better way to hide page elements while rendering
* Room temp and fridge temp are now hidden by default to reduce clutter. Click the circles next to the graph to show them.
* Bug fixes and layout fixes

`All brewpi-www changes on GitHub <https://github.com/BrewPi/brewpi-www/compare/0.2.0...0.3.0>`_

BrewPi Python scripts | 0.2 -> 0.3
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* Changed the way python works with profiles:
    * Support for disabling temperature control in the profile
    * Internal change to the temperature profile format to work with the new interface
* Default first beer is now 'My First BrewPi Run', so it does not append to the sample data
* Added a utils directory with scripts to:
    * Fix permissions
    * Update the CRON job for this version
    * Install all dependencies
    * All of the above: runAfterUpdate.sh
* Resolved startup issues with bootloaders that take longer (stuck at script starting up)
* Added altport setting in config: script will try this alternative port (ttyACM1) wen default port cannot be found
* When restoring settings to the Arduino, send them in a specific order. Fahrenheit settings could be interpreted as Celcius before.
* Commands to start/stop/pause logging (script side)
* Better error exception and lots of bug fixes

`All brewpi-script changes on GitHub <https://github.com/BrewPi/brewpi-script/compare/0.2.0...0.3.0>`_

Arduino code (brewpi-avr)  | 0.2.0 -> 0.2.3
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* Algorithm changes to prevent overshooting beer temperature in fast fridges
    * Immediately stop heating/cooling when beer hits target, regardless of fridge temp/target
    * Reduced minimum on time to 3 minutes (10 minutes for fridge constant cooling to prevent fast cycling)
    * Increased update rate of slope filter, so it has less delay
    * Reduced default PID parameters. A slow controller is better than overshoot
    * Integrator is only updated in IDLE
    * Added tiny idle zone for beer temp (-0.5/+0.5 LSB before filtering)
* Added PID max setting: the max difference between generated fridge setting and beer setting
* Output a new data point on every state transition
* Do not go into heating when no heater is installed
* Time on display is now printed as hours, minutes, seconds: 01h01m39 or 01m39
* Fix for actuators being active for 1 second at boot
* Changed when data is written to EEPROM to reduce number of writes
* Enabled internal pull-ups, so the shield also works well without the display backpack connected.
* Inverted pin mode is now default for new devices
* Beep as first thing at boot, so you know when the bootloader ends and brewpi starts

`All brewpi-avr changes on GitHub <https://github.com/BrewPi/brewpi-avr/compare/0.2.0...0.2.3>`_

BrewPi tools for bootstrapping and updating (brewpi-tools) | New
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* Install script that performs most steps that you previously had to do manually on the command line, mainly:
    * Creating users, directories, etc
    * Installing dependencies (web server, python libraries, etc)
    * Cloning the repositories
    * Setting up the CRON job
* Update script to make it easy to check for updates and apply them
    * Check your configured remotes for updates (not just the official repository)
    * Pull updates from GitHub
    * Ask to stash changes on merge conflicts
    * Switch branches

`All brewpi-tools changes on GitHub <https://github.com/BrewPi/brewpi-tools/compare/master%40%7B5years%7D...0.1.0>`_



