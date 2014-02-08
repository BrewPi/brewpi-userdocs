Programming your Arduino with BrewPi
====================================
Uploading a new hex file to your Arduino can be done straight from the BrewPi web interface.
The serial port is read from the config file (``/home/brewpi/settings/config.cfg``), so make sure the settings are correct.
If this file does not contain port settings, BrewPi uses the defaults from (``/home/brewpi/settings/defaults.cfg``)
BrewPi defaults to ttyACM0 and when it cannot find it, it tries ttyACM1.
If your serial port is one of these, you don't have to do anything.

The default should normally work (``/dev/ttyACM0``), but if your script fails to start because the serial port is not found, run this to see the candidates:

.. code-block:: bash

    ls /dev/ttyA*

It is definitely not ``ttyAMA0``, that is the internal serial port.
Official Arduinos normally appear as ttyACM0 or if that is already taken, ttyACM1.
Arduino clones sometimes appear as ttyUSB0 or similar. If that is the case, add these lines to  (``/home/brewpi/settings/config.cfg``):

.. code-block:: bash

    port = /dev/ttyUSB0
    altport = /dev/ttyUSB1


Uploading a new HEX file to your Arduino
----------------------------------------

To program your Arduino from the web interface, take the following steps:

#.  Log on to your BrewPi web interface by entering the Raspberry Pi's IP address into a web browser.
    The IP is displayed when the installer finishes, or by typing ``ifconfig`` at the shell (look for inet addr).
    If the page says 'script not running' you should start the BrewPi script first.
    This is automatically done by the CRON job within one minute if you click the 'start script' button.
    If you have not set up CRON jet, you can manually start the script with:

    .. code-block:: bash

        sudo -u brewpi python /home/brewpi/brewpi.py

#.  Open the maintenance panel and go to the `Reprogram Arduino` tab.

#.  Download the HEX file appropriate for your setup from http://dl.brewpi.com/brewpi-avr/stable.
    You can also compile your own hex file with Atmel Studio.
    Make sure you have the right file for your Arduino type (UNO or Leonardo) and the right shield (Rev A or Rev C).
    Only the first 100 BrewPi shields were rev A. If you bought one recently, you have Rev C.

#.  The program script can automatically restore your settings and devices after upgrading.
    If you are uploading to a virgin Arduino, just answer NO to both.

#.  Next, just click the program button. If your script was started by CRON, the output will appear in the black box.

#.  The BrewPi script will automatically restart after programming and report which version of BrewPi it has found on the Arduino.


Troubleshooting
---------------
* If you're prompted with an error: "cannot move uploaded file" rerun the fix permissions script in ``sudo sh /home/brewpi/fixPermissions.sh``.
* If saving devices in the device manager does not work, your EEPROM was probably not reset properly. Try reprogramming without settings and devices restore.
* Check whether it is not a hardware problem by programming your Arduino using the Arduino IDE. Just open the `blink` example and click `upload`.


Programming without the web interface
-------------------------------------
If you are having trouble programming through the web interface, you can try running ``programArduinoFirstTime.py``, which is located in your brewpi script directory.
First edit it to have the correct settings. Then run it with:

    .. code-block:: bash

        sudo python /home/brewpi/programArduinoFirstTime.py

Please let us know if you had to resort to this option, it should not be necessary.
