# LIFX Products Description

`products.json` - A JSON representation of the publically available LIFX products.

## Terms and Conditions

LAN Protocol. LiFi Labs, Inc. © 2015. All rights reserved. Usage of this documentation is bound by the [LIFX Developer Terms](http://www.lifx.com/pages/developer-terms-of-use).

### Extended multizone

Multizone products like the LIFX Z and the LIFX Beam have an "Extended Multizone"
api when you have new enough firmware. The version of firmware that enables this
feature is specified by the `min_ext_mz_firmware` and
`min_ext_mz_firmware_components` fields for a multizone product in products.json

These fields refers to the response from a GetHostFirmware packet to your device.
If you interpret the StateHostFirmware packet with `version_major` and
`version_minor` then use the `min_ext_mz_firmware_components` field to make sure
the firmware is new enough. If your StateHostFirmware packet sees just one Uint32
for the version then do the following code to get major and minor and then
also use `min_ext_mz_firmware_components`.

```
major = version >> 0x10
minor = version & 0xFFFF
```

Alternatively you may compare the build timestamp on the StateHostFirmware against
`min_ext_mz_firmware`, but we don't recommend this is as it is a more brittle
comparison.

The comparison logic may look like:

```
desired_major, desired_minor = min_ext_mz_firmware_components
has_extended = device_major >= desired_major and device_minor >= desired_minor
