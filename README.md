# pyevt - A Python API for the Event-Exchanger EVT-2 USB hardware

## 1. About
This repository contains the API to communicate with *EVT-2* USB-devices (+derivatives) developed by the Research Support group of the faculty of Behavioral and Social Science from the University of Groningen. The *EVT-2* is a TTL event marking/TTL triggering device intended for computer-based psychology experiments.

## 2. Dependencies
The *pyevt* API uses [HIDAPI](https://pypi.org/project/hidapi/), a cython module to communicate with HID-class USB devices.

## 3. Install
Install pyevt (and hidapi) with:

`pip install pyevt` or

`pip install --user pyevt` on managed computers.

## 4. Device Permission for Linux
In Linux, python HIDAPI uses the *libusb* backend (not *hidraw*). After connecting the EVT device, check for the EVT idVendor:idProduct ID's with:

`lsusb`

Permission for EVT devices should be given by adding the next lines to a udev file, for example:

In `/etc/udev/rules.d` create a device rules file with:
`sudo touch 99-evt-devices.rules`

Edit this file with: `sudo nano 99-evt-devices.rules` and add the following lines:

```
# /etc/udev/rules.d/99-evt-devices.rules

# Used EVT device
SUBSYSTEM=="usb", ATTRS{idVendor}=="0808", ATTRS{idProduct}=="0002", GROUP="plugdev", MODE="0660"
```

Save and close `99-evt-devices.rules`

The user should retrigger the udev rules after disconnecting the EVT-device first with:

`sudo udevadm control --reload-rules && sudo udevadm trigger`

The user should be a member of the `plugdev` -group.

Check with:

`groups username`

If this is not the case, add the user to the `plugdev` group by typing:

`sudo usermod -a -G plugdev username`

Plugin the EVT-device and ready you are.

## 5. EventExchanger Class — API Summary

**Device Management**

| Method                                       | Description                                                                                                  |
| -------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| `scan(matching_key="EventExchanger")`        | Lists connected HID devices whose product name contains `matching_key`. Returns a list of device info dicts. |
| `attach_name(matching_key="EventExchanger")` | Attaches the first device whose product name contains `matching_key`.                                        |
| `attach_id(path)`                            | Attaches a device using its unique HID `path`.                                                               |
| `close()`                                    | Closes the currently attached device.                                                                        |
| `reset()`                                    | Sends a reset command to the device and disconnects it from USB.                                             |

**Event & Data Retrieval**

| Method                                            | Description                                                                                                                       |
| ------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| `wait_for_event(allowed_event_lines, timeout_ms)` | Waits for digital input events matching the given bit mask. Returns `(event_code, elapsed_ms)`, or `(-1, elapsed_ms)` on timeout. |
| `get_axis()`                                      | Reads the current axis position value from the device.                                                                            |

**Digital Output Control**

| Method                            | Description                                                     |
| --------------------------------- | --------------------------------------------------------------- |
| `write_lines(value)`              | Sets digital output lines to the specified bit pattern (0–255). |
| `pulse_lines(value, duration_ms)` | Pulses output lines for the given duration in milliseconds (0-65535).     |
| `clear_lines()`                   | Clears (sets low) all output lines.                             |


**Analog & Encoder Control**

| Method                                                                       | Description                               |
| ---------------------------------------------------------------------------- | ----------------------------------------- |
| `set_analog_event_step_size(samples_per_step)`                               | Configures analog event step size.        |
| `renc_init(encoder_range, min_value, position, input_change, pulse_divider)` | Initializes rotary encoder parameters.    |
| `renc_set_pos(position)`                                                     | Sets the current rotary encoder position. |


**LED Control**

| Method                                            | Description                         |
| ------------------------------------------------- | ----------------------------------- |
| `set_led_rgb(red, green, blue, led_number, mode)` | Sets RGB color for a specific LED.  |
| `send_led_rgb(num_leds, mode)`                    | Sends LED color data to the device. |


## 6. Python coding examples

```
from pyevt import EventExchanger

myevt = EventExchanger()
# Get list of devices containing the partial string 'partial_device_name'
myevt.scan('partial_device_name') # The default is 'EventExchanger'.

# Create a device handle:
myevt.attach_name('partial_device_name') # Example: 'EVT02', 'SHOCKER' or 'RSP-12', etc. The default is 'EventExchanger'.

myevt.write_lines(0) # clear outputs
myevt.pulse_lines(170, 1000) # value=170, duration=1000ms

# remove device handle
myevt.close()

# connect RSP-12 button response box
myevt.attach_name('RSP-12')
myevt.wait_for_event(3, None) # wait for button 1 OR 2, timeout is infinite.
myevt.close() # remove device handle

```

## 7. License
The pyevt API is distributed under the terms of the GNU General Public License 3.
The full license should be included in the file COPYING, or can be obtained from

[http://www.gnu.org/licenses/gpl.txt](http://www.gnu.org/licenses/gpl.txt)

The pyevt API contains the work of others.

## 8. Other documentation
Information about EVT-devices and OpenSesame plugins:

[https://markspan.github.io/evtplugins/](https://markspan.github.io/evtplugins/)
