---
# victron_ble

A Python library to parse Instant Readout advertisement data from Victron devices.

Disclaimer: This software is not an officially supported interface by Victron and is provided entirely "as-is"

**Supported Devices:**

* SmartShunt 500A/500mv (and likely BMV-712/702) provide the following data:
    * Voltage
    * Alarm status
    * Current
    * Remaining time
    * State of charge (%)
    * Consumed amp hours
    * Auxillary input (temperature, midpoint voltage, or starter battery voltage)
* Smart Battery Sense
    * Voltage
    * Temperature (°C)

If you'd like to support development for additional devices, consider [sponsoring this project](https://github.com/sponsors/keshavdv/)


## Install it from PyPI

```bash
pip install victron_ble
```

## Usage

To be able to decrypt the contents of the advertisement, you'll need to first fetch the per-device encryption key from the official Victron application. The method to do this will vary per platform, but only instructions for the OSX app are provided below for now: 

#### Fetching Keys
 
**OSX**

1. Install the Victron app from the Mac App Store
2. Pair with your device at least once to transfer keys
3. Run the following from Terminal to dump the known keys (install sqlite3 via Homebrew)
```bash
❯ sqlite3 ~/Library/Containers/com.victronenergy.victronconnect.mac/Data/Library/Application\ Support/Victron\ Energy/Victron\ Connect/d25b6546b47ebb21a04ff86a2c4fbb76.sqlite 'select address,advertisementKey from advertisementKeys inner join macAddresses on advertisementKeys.macAddress == macAddresses.macAddress'

{763aeff5-1334-e64a-ab30-a0f478s20fe1}|0df4d0395b7d1a876c0c33ecb9e70dcd
❯
````

#### Reading data

The project ships with a standalone CLI that can be used to print device data to the console. 

```bash
# Will show all discovered Victron devices with Instant Readout enabled, their names, and IDs
$ > victron-ble discover 
763aeff5-1334-e64a-ab30-a0f478s20fe1: SmartShunt HT4531A246S
...


# Dump data for a particular device (replace the ID and key with your own)
$ > victron-ble read "763aeff5-1334-e64a-ab30-a0f478s20fe1:0df4d0395b7d1a876c0c33ecb9e70dcd"
INFO:victron_ble.scanner:Reading data for ['763aeff5-1334-e64a-ab30-a0f478s20fe1']
{
  "name": "SmartShunt HT4531A246S",
  "address": "763AEFF5-1334-E64A-AB30-A0F478S20FE1",
  "rssi": -79,
  "payload": {
    "remaining_mins": 65535,
    "aux_mode": 3,
    "aux": 0,
    "current": 0,
    "voltage": 12.52,
    "consumed_ah": 50.0,
    "soc": 50.0,
    "alarm": {
      "low_voltage": false,
      "high_voltage": false,
      "low_soc": false,
      "low_starter_voltage": false,
      "high_starter_voltage": false,
      "low_temperature": false,
      "high_temperature": false,
      "mid_voltage": false
    }
  }
}
...

# Dump data for debugging and supporting new devices (replace the ID)
$ > victron-ble dump "763aeff5-1334-e64a-ab30-a0f478s20fe1"
Dumping advertisements from 763aeff5-1334-e64a-ab30-a0f478s20fe1
1671843194.0534039      : 100289a302413bafd03bb245e131ae926267f6fd0b59e0
1671843194.682535       : 100289a302423baf58a1546e5262dcdf0ef642f353ed65
1671843197.676384       : 100289a302453baf804707549cffb2ab970c981ae897b6
...
```

To consume this project as a library, you can import the particular parser for your device:
```py
from victron_ble.devices import detect_device_type

data = <ble advertisement data>
parser = detect_device_type(data)
parsed_data = parser(<key>).parse(<ble advertisement data>)
```

## Development

If you'd like to help support a new device, collect the following and create a new Github issue:

1. Run `victron-ble discover` to find the ID of the device you'd like to support
2. Run `victron-ble dump <ID>` for an few minutes while collecting corresponding screenshots from the official apps instant readout to identify the current values

For pull requests:

Read the [CONTRIBUTING.md](CONTRIBUTING.md) file.

## Contributors

Special thanks to https://github.com/rochacbruno/python-project-template for the project template