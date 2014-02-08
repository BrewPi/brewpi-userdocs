Changelog
=========
February 7th, 2014
------------------
* Changed how the CRON job is managed. Moved all functionality into updateCron.sh.
* Fixed the CRON entry for wifiChecker.sh


January 20th, 2014
------------------
In this release: a new legend for the charts, mash temperatures,

Web interface (brewpi-www) | 0.3.0 -> 0.3.1
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Brian Schwinn reworked the legend for the beer charts to show much more info and to be a lot prettier.

*New legend for the beer chart*

* Legend moved to the right instead of on top of the chart.
* Legend now shows in which state BrewPi was (cooling/heating/waiting etc.) on mouse-over
* Enabling/disabling lines in the chart (click the legend item) is now stored in HTML5 Local Storage, so it is remembered when you reload the page.

*Various bug fixes and small edits:*

* User documentation removed from brewpi-www repo and moved to separate repository (brewpi-userdocs)
* Beer name removed from maintenance panel (start/stop beer is now done by clicking beer name)
* Added a refresh button if the chart has no data points yet
* Fix for DS2413 devices in device manager
* Better error checking when loading logs
* Removed 'profiles' directory from previous beers drop-down menu

`All brewpi-www changes on GitHub <https://github.com/BrewPi/brewpi-www/compare/0.3.0...0.3.1>`_

BrewPi Python scripts | 0.3.0 -> 0.3.1
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Geo did most work for the BrewPi script this time, he added a script to keep Wifi alive and reworked the update script to also update the Arduino.

*WiFi Checker Script*

A new script in the utils directory can check whether the WiFi connection is working properly and if not, it resets wlan0 using ifup and ifdown.
This script can be installed using 'checkScript.sh install'. This adds it to cron.d/brewpi to run every 10 minutes.

*Small changes*:

* Moved opening serial into BrewPiUtil, so it can be used by the programming script too.
* Added pin list for DIY shield
* Parsing of the commit SHA in the Arduino version string

`All brewpi-script changes on GitHub <https://github.com/BrewPi/brewpi-script/compare/0.3.0...0.3.1>`_


Arduino code (brewpi-avr)  | 0.2.3.1 -> 0.2.4
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The biggest changes in the Arduino code for this release are:

* Shift of the internal temperature format from -64/+64 degree Celsius to -16/+112 degree Celsius. BrewPi can now be used to log your mash temperatures too (Elco).
  Actuators with PWM for mashing will come in the next release.
* Better handling of temperature sensor disconnects and resets (awesome work by Matthew McGowan).
* Added a NetBeans project to build the Arduino code into a standalone windows application (exe), for testing and simulation (Matthew)
* Added googletest framework for unit testing (Matthew)

*Sensor reset detection and smart initialization (Matthew)*

* Sensor reset detection: the DS18B20 loads values from non-volatile (EEPROM) memory to volatile memory on startup.
  By writing a different value to the volatile memory after init, we can see when the sensors have been reset between reads.
  When a reset is detected, we know not to thrust the next read form the sensor (which defaults to 85C)
* Removed unused fields and code in the DallasTemperature library for reduced code size and memory footprint.
* Smarter re-initialization of sensors and filters. It is now only done after a number of failed reads.


*Various changes*

* Changed receive function to handle all incoming messages instead of one each main loop. This bug should have been noticed much earlier.
  It could cause the Arduino to be slow to respond to messages in certain circumstances.
* Included Arduino files in the project and removed USB HID driver support from Arduino files. This saves 918 bytes!
* Changed how the processor/Arduino type is detected in the build (now based on processor)
* Removed devices installed by default for RevA shields. These caused conflicts when devices were restored too. Now RevA behaves like RevC.
* Support for using DS2413 as switch input (Matthew)
* Added a minimum for overshoot estimators. They could not recover from being zero before this fix.
* Include commit SHA in version number

`All brewpi-www changes on GitHub <https://github.com/BrewPi/brewpi-www/compare/0.2.3.1...0.2.4>`_

Install and update tools (Brewpi-tools)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* Update script now also checks Arduino version and can reprogram the Arduino. It downloads the latest hex file from the BrewPi server.
* Added Wifi check script to install (add to cron.d)

`All brewpi-tools changes on GitHub <https://github.com/BrewPi/brewpi-tools/compare/0.1.0...0.2.0>`_


December 23, 2013
-----------------

Arduduino code(brewpi-avr) | 0.2.3 -> 0.2.3.1
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Hotfix: pidMax was printed as a temperature (with offset in Fahrenheit) which caused it to be restored incorrectly when programming the Arduino.


October 22, 2013
------------------
It's been a long time since a master release, but in this release we made some major steps to make releasing easier in the future.

The biggest changes in this release are:

* Added a native interface for beer profiles. No more google spreadsheets!
  Huge thank you to Brian Schwinn (bschwinn) for his hard work on this feature.
* Added an easy menu to start a new brew, start/stop/pause data logging
* Added an install and update script! Just run the script and afterwards go straight to flashing your Arduino and setting up devices.
  Huge thank you to Geo van O. for his hard work on these scripts.
* Tweaked the temperature control algorithm to reduce overshoot.
* Use cron.d instead of crontab to make automated updating of the cron job easier

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



