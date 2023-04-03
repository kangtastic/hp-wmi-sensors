.. SPDX-License-Identifier: GPL-2.0-or-later

Linux HP WMI Sensors Driver
===========================

Description
-----------

Hewlett-Packard business-class computers report hardware monitoring information
via Windows Management Instrumentation (WMI). This driver exposes that
information to the Linux ``hwmon`` subsystem, allowing familiar userspace
utilities like ``sensors`` to gather numeric sensor readings.

Installation, uninstallation, and usage
---------------------------------------

Assuming the existence of a relatively up-to-date development environment
suitable for building Linux kernel modules, including kernel headers, a GCC
compiler toolchain, and DKMS, simply issuing::

    $ sudo make dkms

should be sufficient to build and install the driver, and::

    $ sudo make dkms_clean

should likewise be sufficient to uninstall it.

Once installed, the driver may optionally be set to load automatically upon
system startup. Refer to the documentation for your Linux distribution for
specific instructions.

``sysfs`` interface
-------------------

When the driver is loaded, it discovers the sensors available on the current
system and creates the following read-only ``sysfs`` attributes as appropriate
within ``/sys/class/hwmon/hwmonX``:

================ ===================================
Name		 Description
================ ===================================
curr[X]_input    Current in milliamperes (mA).
curr[X]_label    Current sensor label.
fan[X]_input     Fan speed in RPM.
fan[X]_label     Fan sensor label.
fan[X]_fault     Fan sensor fault indicator.
in[X]_input      Voltage in millivolts (mV).
in[X]_label      Voltage sensor label.
temp[X]_input    Temperature in millivolts (mV).
temp[X]_label    Temperature sensor label.
temp[X]_fault    Temperature sensor fault indicator.
================ ===================================

Here, ``X`` is some number that depends on other available sensors and on other
system hardware components.

``fault`` attributes
  Reading ``1`` instead of ``0`` as the ``fault`` attribute for a sensor
  indicates that the sensor has encountered some issue during operation, and
  that measurements from it should no longer be trusted.

``debugfs`` interface
---------------------

The standard ``hwmon`` interface in ``sysfs`` exposes sensors of several common
types that are connected and operating normally as of driver initialization.
However, there are usually other sensors on the WMI side that do not meet these
criteria. This driver therefore provides a ``debugfs`` interface in
``/sys/kernel/debug/hp-wmi-sensors-X`` that allows read-only access to *all* HP
WMI sensors on the current system.

.. warning:: The ``debugfs`` interface is only available when the kernel is
             compiled with option ``CONFIG_DEBUG_FS``, and its implementation
             is subject to change without notice at any time.

One numbered entry is created per sensor with the following attributes:

=============================== ==========================================
Name				Example
=============================== ==========================================
name                            ``CPU0 Fan``
description                     ``Reports CPU0 fan speed``
sensor_type                     ``12``
other_sensor_type               ``(null)``
operational_status              ``2``
current_state                   ``Normal``
possible_states                 ``Normal\nCaution\nCritical\nNot Present``
base_units                      ``19``
unit_modifier                   ``0``
current_reading                 ``1008``
=============================== ==========================================

These represent the properties of the underlying ``HP_BIOSNumericSensor`` WMI
object, some of which may vary in contents and formatting (but not presence or
semantics) between systems. See [#]_ for more details.

Known issues and limitations
----------------------------

- Non-numeric HP sensor types such as intrusion sensors that belong to the
  ``HP_BIOSStateSensor`` WMI object type are not supported.
- It is intended that the ``debugfs`` interface will facilitate supporting more
  types in the future. Whether systems that actually implement more than the
  types already supported exist in the wild is unknown.

Acknowledgements
----------------

Portions of the code are based on ``asus-wmi-sensors`` [#]_ (@electrified)
and ``corsair-psu`` [#]_ (@wgottwalt).

We sincerely thank the authors and maintainers of those projects for their
exemplary contributions to the Linux community.

References
----------

.. [#] Hewlett-Packard Development Company, L.P.,
       "HP Client Management Interface Technical White Paper", 2005. [Online].
       Available: https://h20331.www2.hp.com/hpsub/downloads/cmi_whitepaper.pdf

.. [#] https://github.com/electrified/asus-wmi-sensors

.. [#] https://github.com/wgottwalt/corsair-psu
