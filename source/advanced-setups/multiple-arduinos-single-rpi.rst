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

You will get a bunch of output.  The relevant ``looking at parent device`` section will be the one that contains a line that states ``ATTRS{product}=="Arduino Leonardo"`` (your Arduino model may differ).  Here is the output from one of my Arduinos:

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

+-------------------------------+-----------------------------------------------------------------------------+
| Parameter                     | What to set it to                                                           |
+===============================+=============================================================================+
| ``KERNELS=="1-1.3.3.3"``      | | The identifier from the previous section for the port you're working with |
+-------------------------------+-----------------------------------------------------------------------------+
| ``SYMLINK+="arduino_bottom"`` | | The name of the symlink you wish to create in ``/dev/``.                  |
|                               | | Do not inclue the leading ``/dev/``.                                      |
+-------------------------------+-----------------------------------------------------------------------------+

In the example above, I end up with the symlinks ``/dev/arduino_bottom`` and ``/dev/arduino_top`` when both Arduinos are connected to their respective ports.

Once the udev rules file is created, disconnect your Arduino and then reload udev before connecting all of the Ardiunos to their respective ports.

.. code-block:: bash

    sudo /etc/init.d/udev reload

Install the BrewPi script
-------------------------



.. [#] `How to distinguish between identical USB-to-serial adapters? - Ask Ubuntu <http://askubuntu.com/a/50412>`_
