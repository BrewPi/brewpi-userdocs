Running Multiple Arduinos on the Same RaspberryPi
=================================================
In some cases it may be desired to run multiple Arduinos on the same RaspberryPi (RPi).  A common reason for doing is to support multiple chambers (one Arduino per chamber) off a single RPi.

This guide explains how to configure your RPi to run two instances of BrewPi to support two Arduinos controlling two chambers ('top' and 'bottom').  The use of the `manual installation process <../manual-brewpi-install/manual-brewpi-install.rst>`_ is necessary.

Setup udev rules for the Arduinos
---------------------------------
By using udev, you can create static device node symlinks based on the port of the USB hub that each Arduino is plugged into.[#]  This allows you to be confident that each script instance is always controlling the correct Arduino.

Determine the USB hub port identifier
"""""""""""""""""""""""""""""""""""""
With only one Arduino connected, issue the following command to see which device node the Arduino is currently using:

.. code-block:: bash

    ls /dev/ttyACM*

Then issue the following command, using the device node name:

.. code-block:: bash

    udevadm info -a -n /dev/arduino_bottom | less

You will get a bunch of output.  The relevant ``looking at parent device`` section will be the one that contains a line that states ``ATTRS{product}=="Arduino Leonardo"`` (your Arduino model may differ).  Here is the output from the section to look at from one of my Arduinos:

.. code-block:: bash

      looking at parent device '/devices/platform/bcm2708_usb/usb1/1-1/1-1.3/1-1.3.3/1-1.3.3.3':
        KERNELS=="1-1.3.3.3"
        SUBSYSTEMS=="usb"
        DRIVERS=="usb"
        ATTRS{bDeviceSubClass}=="00"
        ATTRS{bDeviceProtocol}=="00"
        ATTRS{devpath}=="1.3.3.3"
        ATTRS{idVendor}=="2341"
        ATTRS{speed}=="12"
        ATTRS{bNumInterfaces}==" 3"
        ATTRS{bConfigurationValue}=="1"
        ATTRS{bMaxPacketSize0}=="64"
        ATTRS{busnum}=="1"
        ATTRS{devnum}=="9"
        ATTRS{configuration}==""
        ATTRS{bMaxPower}=="500mA"
        ATTRS{authorized}=="1"
        ATTRS{bmAttributes}=="80"
        ATTRS{bNumConfigurations}=="1"
        ATTRS{maxchild}=="0"
        ATTRS{bcdDevice}=="0100"
        ATTRS{avoid_reset_quirk}=="0"
        ATTRS{quirks}=="0x0"
        ATTRS{version}==" 2.00"
        ATTRS{urbnum}=="19"
        ATTRS{ltm_capable}=="no"
        ATTRS{manufacturer}=="Arduino LLC"
        ATTRS{removable}=="unknown"
        ATTRS{idProduct}=="8036"
        ATTRS{bDeviceClass}=="00"
        ATTRS{product}=="Arduino Leonardo"

The important line in the output above is the ``KERNELS`` line.  Write that line down and save it for later.

.. code-block:: bash

    KERNELS=="1-1.3.3.3"

Next, move the Arduino to the next port you will use for another Arduino and repeat the steps previously listed in this section.  Repeat the process for as many Arduinos as you will be using.

Write the udev rules
""""""""""""""""""""
Now that the identifier for each USB hub port has been obtained, the udev rules can be written.  In my case, I have the following identifiers:

.. code-block:: bash

    KERNELS=="1-1.3.3.3"
    KERNELS=="1-1.3.3.4"

Create the file ``/etc/udev/rules.d/99-arduino.rules`` with contents similar to the following:

.. code-block:: bash

    SUBSYSTEM=="tty", KERNEL=="ttyACM*", KERNELS=="1-1.3.3.3", SYMLINK+="arduino_bottom"
    SUBSYSTEM=="tty", KERNEL=="ttyACM*", KERNELS=="1-1.3.3.4", SYMLINK+="arduino_top"

The parameters on each line to change are the following:

+-------------------------------+----------------------------------------------------------------------------------------------------+
| Parameter                     | Value                                                                                              |
+===============================+====================================================================================================+
| ``KERNELS=="1-1.3.3.3"``      | | Set to the identifier from the previous section that corresponds to the port you're working with |
+-------------------------------+----------------------------------------------------------------------------------------------------+
| ``SYMLINK+="arduino_bottom"`` | | Set to the name of the symlink you wish to create in ``/dev/``.                                  |
|                               | | Do not inclue the leading ``/dev/``.                                                             |
+-------------------------------+----------------------------------------------------------------------------------------------------+

In the example above, I end up with the symlinks ``/dev/arduino_bottom`` and ``/dev/arduino_top`` when both Arduinos are connected to their respective ports.  The symlink names reflect which chamber each Arduino controls.

Once the udev rules file is created, disconnect your Arduino and then reload udev before connecting all of the Ardiunos to their respective ports.

.. code-block:: bash

    sudo /etc/init.d/udev reload

Install BrewPi
--------------
Install the BrewPi script manually as described in the `manual installation process`_, noting the following changes:

* ``git clone`` brewpi-script into subdirectories of ``/home/brewpi`` instead of directly into ``/home/brewpi``.  I used ``/home/brewpi/top`` and ``/home/brewpi/bottom`` to match the chamber each Arduino controls.
* ``git clone`` brewpi-www into subdirectories of ``/var/www`` instead of directly into ``/var/www``.  I used ``/var/www/top`` and ``/var/www/bottom`` to match each script installation directory.
* Fix the permissions manually.

  * **UNTESTED** alternative
  
    * It looks like ``utils/fixPermissions.sh`` should work when run from each script instance.
    * If you have other content in ``/var/www``, you will likely want to update ``webPath`` in ``fixPermissions.sh`` to the installation directory of the matching web interface instance.

* Do  **not** use ``utils/updateCron.sh`` or the cron job string in the manual install instructions.  Instead follow the directions in the cron section below.

Modify the config files
-----------------------

Edit the script config files
""""""""""""""""""""""""""""
``settings/config.cfg`` needs to be created in each script instance to properly configure them.  Here are the config files I'm using.

/home/brewpi/bottom/settings/config.cfg
'''''''''''''''''''''''''''''''''''''''

.. code-block:: bash

    scriptPath = /home/brewpi/bottom/
    wwwPath = /var/www/bottom/
    port = /dev/arduino_bottom
    altport = /dev/null
    boardType = leonardo

/home/brewpi/top/settings/config.cfg
''''''''''''''''''''''''''''''''''''

.. code-block:: bash

    scriptPath = /home/brewpi/top/
    wwwPath = /var/www/top/
    port = /dev/arduino_top
    altport = /dev/null
    boardType = leonardo

Variable explanation
''''''''''''''''''''

+------------+------------------------------------------------------------------------------------------------------------------------------------------------------+
| Variable   | Value                                                                                                                                                |
+============+======================================================================================================================================================+
| scriptPath | | Set to the full path of this script instance.  Include the trailing slash.                                                                         |
+------------+------------------------------------------------------------------------------------------------------------------------------------------------------+
| wwwPath    | | Set to the full path of the web interface instance that corresponds to this script instance.  Include the trailing slash.                          |
+------------+------------------------------------------------------------------------------------------------------------------------------------------------------+
| port       | | Set to the device node symlink for the Arduino that corresponds to this script instance.  This symlink was set up in the udev rules section above. |
+------------+------------------------------------------------------------------------------------------------------------------------------------------------------+
| altport    | | Set to ``/dev/null`` so that the default alternate port will not be attempted.                                                                     |
|            | | Because the device node symlink will always be correct, you don't want an alternate port to be used                                                |
+------------+------------------------------------------------------------------------------------------------------------------------------------------------------+
| boardType  | | Set to your Arduino board type.                                                                                                                    |
+------------+------------------------------------------------------------------------------------------------------------------------------------------------------+

Edit the web interface config files
"""""""""""""""""""""""""""""""""""
``config_user.php`` needs to be created in each web interface instance to properly configure them.  Here are the config files I'm using.

/var/www/bottom/config_user.php
'''''''''''''''''''''''''''''''

.. code-block:: bash

    <?php
            // The default settings in config.php are overruled by the settings in config_user.php
            // To use custom settings, copy this file to config_user.php and make your changes in config_user.php
            // do not add a php closing tag, because newlines after closing tag might be included in the html

            // Do not include a trailing slash on the path
            $scriptPath = '/home/brewpi/bottom';

/var/www/top/config_user.php
''''''''''''''''''''''''''''

.. code-block:: bash

    <?php
            // The default settings in config.php are overruled by the settings in config_user.php
            // To use custom settings, copy this file to config_user.php and make your changes in config_user.php
            // do not add a php closing tag, because newlines after closing tag might be included in the html

            // Do not include a trailing slash on the path
            $scriptPath = '/home/brewpi/top';

Variable explanation
''''''''''''''''''''

+-------------+--------------------------------------------------------------------------------------------------------------------------------------+
| Variable    | Value                                                                                                                                |
+=============+======================================================================================================================================+
| $scriptPath | | Set to the full path of the script instance that corresponds to this web interface instance.  Do **not** include a trailing slash. |
+-------------+--------------------------------------------------------------------------------------------------------------------------------------+

Set up cron jobs to start the scripts
-------------------------------------
Create cron job files for each script instance.  Here are the config files I'm using.

/etc/cron.d/brewpi_bottom
'''''''''''''''''''''''''

.. code-block:: bash

    PYTHON=/usr/bin/python
    SCRIPTPATH=/home/brewpi/top

    * * * * * brewpi $PYTHON $SCRIPTPATH/brewpi.py --config $SCRIPTPATH/settings/config.cfg --checkstartuponly --dontrunfile; [ $? != 0 ] && $PYTHON -u $SCRIPTPATH/brewpi.py --config $SCRIPTPATH/settings/config.cfg 1>$SCRIPTPATH/logs/stdout.txt 2>>$SCRIPTPATH/logs/stderr.txt &

/etc/cron.d/brewpi_top
''''''''''''''''''''''

.. code-block:: bash

    PYTHON=/usr/bin/python
    SCRIPTPATH=/home/brewpi/top

    * * * * * brewpi $PYTHON $SCRIPTPATH/brewpi.py --config $SCRIPTPATH/settings/config.cfg --checkstartuponly --dontrunfile; [ $? != 0 ] && $PYTHON -u $SCRIPTPATH/brewpi.py --config $SCRIPTPATH/settings/config.cfg 1>$SCRIPTPATH/logs/stdout.txt 2>>$SCRIPTPATH/logs/stderr.txt &

Variable explanation
''''''''''''''''''''

+------------+------------------------------------------------------------------------------------------------------------------------+
| Variable   | Value                                                                                                                  |
+============+========================================================================================================================+
| PYTHON     | | Set to the full path of the Python binary                                                                            |
+------------+------------------------------------------------------------------------------------------------------------------------+
| SCRIPTPATH | | Set to the full path of the script instance that corresponds to this cron job.  Do **not** include a trailing slash. |
+------------+------------------------------------------------------------------------------------------------------------------------+

``--config $SCRIPTPATH/settings/config.cfg`` is specified for both invocations of the script in the cron job so that BrewPi's process monitoring can see that each script instance is unique.  For a description of the rest of the items in the cron job command, see the `manual installation process cron job page <../manual-brewpi-install/setting-up-cron.rst>`_.

Updating
--------
I have not investigated whether it is safe to use the updater script from ``brewpi-tools``, so at this point I would recommend doing updates manually.

References
----------
.. [#] `How to distinguish between identical USB-to-serial adapters? - Ask Ubuntu <http://askubuntu.com/a/50412>`_
