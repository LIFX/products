# LIFX Products Description

`products.json` - A JSON representation of the publically available LIFX products.

## Terms and Conditions

LAN Protocol. LiFi Labs, Inc. © 2021. All rights reserved. Usage of this documentation is bound by the [LIFX Developer Terms](http://www.lifx.com/pages/developer-terms-of-use).

### Available capabilities

* `hev` = The light supports emitting HEV light
* `color` = The light changes physical appearance when the Hue value is changed
* `chain` = The light may be connected to physically separated hardware
  (currently only the LIFX Tile)
* `matrix` = The light supports a 2D matrix of LEDs (the Tile and Candle)
* `relays` = The device has relays for controlling physical power to something
  (the LIFX Switch)
* `buttons` = The device has physical buttons to press (the LIFX Switch)
* `infrared` = The light supports emitting infrared light
* `multizone` = The light supports a 1D linear array of LEDs (the Z and Beam)
* `temperature_range` = An array of the minimum and maximum kelvin values this
  device supports. If the numbers are the same then the device does not support
  variable kelvin values. It is `null` for devices that aren't lighting
  products (the LIFX Switch)
* `extended_multizone` = The more capable `extended` API for multizone control
  that lets us control all the zones on the device with a single message instead
  of many.

### Determining capabilities

To determine the capabilities of a device you need four values from the device:

* `vendor_id`, this will likely be `1` which says it's a LIFX device
* `product_id`, this has a different number per product
* `firmware_major`, the major revision number for the firmware
* `firmware_minor`, the minor revision number for the firmware

You can get the first two numbers via the `GetVersion` message, and the second
two via the `GetHostFirmware` message.

So for example, if I had a candle on `(3, 60)` firmware I'd have
`vid:1 pid:57 major:3 minor:60`.

We can then create a dictionary of capabilities from the json using code that
looks like this.

```python
def get_capabilities(vid, pid, major, minor):
    for by_vendor in products:
        if by_vendor["vid"] != vid:
            continue

        cap = by_vendor["default"]

        for product in by_vendor["products"]:
            if product["pid"] != pid:
                continue

            cap.update(product["features"])

            for upgrade in product["upgrades"]:
                if (ma, mi) >= (upgrade["major"], upgrade["minor"]):
                    cap.update(upgrade["features"])

            return cap
```

Note that this method will replace the need to look for `min_ext_mz_firmware` and
`min_ext_mz_firmware_components` as it is replaced with the `extended_multizone`
capability that becomes true after a new enough firmware.

### Firmware Major/Minor

Some functionality in devices exists only after certain firmware versions and so
it can be necessary to look the firmware on the device to determine if that device
can do particular things.

LIFX firmware is identified by a pair of numbers, referred to as the `major` and
`minor` versions. For example, `(3, 60)` or `(2, 80)`.

The `major` component will stay consistent over time for different generations of
our hardware, whilst the `minor` component is incremented.

These fields refers to the response from a GetHostFirmware packet to your device.
Our public protocol has these two components as separate fields in the packet
description, but there was a time when it was described using one `Uint32`.

For libraries that have this, you may use the following code to split that number:

```
major = version >> 0x10
minor = version & 0xFFFF
```

Once you have your two components, you can compare a desired pair with your
retrieved pair with something like:

```
has_capability = device_major >= desired_major and device_minor >= desired_minor
```

In Python, you can compare two tuples for the same affect:

```
has_capability = (device_major, device_minor) >= (desired_major, desired_minor)
```

Determining Extended Multizone
------------------------------

Before changes made in February 2021 that describes the process above, we had
two fields to determine if a device has new enough firmware to support our
`extended multizone` messages. These were `min_ext_mz_firmware_components` and
`min_ext_mz_firmware`.

If you are working with a version of the `products.json` that does not contain
the `upgrades` structure, you may use `min_ext_mz_firmware` as the `(major, minor)`
pair for comparison and safely ignore `min_ext_mz_firmware`.
