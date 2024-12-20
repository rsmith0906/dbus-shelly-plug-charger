# dbus-shelly-plug-charger
Integrate Shelly Plug into Victron Energies Venus OS

## Purpose
With the scripts in this repo it should be easy possible to install, uninstall, restart a service that connects the Shelly Plug to the VenusOS and GX devices from Victron.
Idea is inspired on @vikt0rm project linked below.

## Inspiration
This project with the Victron Venus OS, so I took some ideas and approaches from the following projects - many thanks for sharing the knowledge:
- https://github.com/vikt0rm/dbus-shelly-1pm-pvcharger

## How it works
### My setup
- 1-Phase installation
- Shelly Plug with latest firmware
  - Measuring AC output of Off-Grid charger
  - Connected to Wifi netowrk "A" with a known IP  
- Venus OS on Raspberry PI 3 4GB
  - Victron 150/45 Solar Charge Controller
  - Victron SmartShunt
  - Connected to Wifi netowrk "A"

### Details / Process
As mentioned above the script is inspired by @vikt0rm dbus-shelly-1em-pvcharger implementation.
So what is the script doing:
- Running as a service
- connecting to DBus of the Venus OS `com.victronenergy.charger.http_{DeviceInstanceID_from_config}`
- After successful DBus connection Shelly Plug is accessed via REST-API - simply the /status is called and a JSON is returned with all details
  A sample JSON file from Shelly 1PM can be found [here](docs/shelly1pm-status-sample.json)
- Serial/MAC is taken from the response as device serial
- Paths are added to the DBus with default value 0 - including some settings like name, etc
- After that a "loop" is started which pulls Shelly Plug data every 750ms from the REST-API and updates the values in the DBus

Thats it 😄

### Pictures
![Tile Overview](img/venus-os-tile-overview.PNG)
![Remote Console - Overview](img/venus-os-remote-console-overview.PNG) 
![SmartMeter - Values](img/venus-os-shelly1pm-pvcharger.PNG)
![SmartMeter - Device Details](img/venus-os-shelly1pm-pvcharger-devicedetails.PNG)


## Install & Configuration
### Get the code
Just grap a copy of the main branche and copy them to a folder under `/data/` e.g. `/data/dbus-shelly-plug-charger`.
After that call the install.sh script.

The following script should do everything for you:
```
wget https://github.com/rsmith0906/dbus-shelly-plug-charger/archive/refs/heads/main.zip
unzip main.zip "dbus-shelly-plug-charger-main/*" -d /data
mv /data/dbus-shelly-plug-charger-main /data/dbus-shelly-plug-charger
chmod a+x /data/dbus-shelly-plug-charger/install.sh
/data/dbus-shelly-plug-charger/install.sh
rm main.zip
```
⚠️ Check configuration after that - because service is already installed an running and with wrong connection data (host, username, pwd) you will spam the log-file

### Change config.ini
Within the project there is a file `/data/dbus-shelly-plug-charger/config.ini` - just change the values - most important is the deviceinstance, custom name and phase under "DEFAULT" and host, username and password in section "ONPREMISE". More details below:

| Section  | Config vlaue | Explanation |
| ------------- | ------------- | ------------- |
| DEFAULT  | AccessType | Fixed value 'OnPremise' |
| DEFAULT  | SignOfLifeLog  | Time in minutes how often a status is added to the log-file `current.log` with log-level INFO |
| DEFAULT  | Deviceinstance | Unique ID identifying the shelly 1pm in Venus OS |
| DEFAULT  | CustomName | Name shown in Remote Console (e.g. name of pv charger) |
| DEFAULT  | Phase | Valid values L1, L2 or L3: represents the phase where pv charger is feeding in |
| DEFAULT  | Position | Valid values 0, 1 or 2: represents where the charger is connected (0=AC input 1; 1=AC output; 2=AC input 2) |
| ONPREMISE  | Host | IP or hostname of on-premise Shelly 3EM web-interface |
| ONPREMISE  | Username | Username for htaccess login - leave blank if no username/password required |
| ONPREMISE  | Password | Password for htaccess login - leave blank if no username/password required |



## Used documentation
- https://github.com/victronenergy/venus/wiki/dbus#pv-chargers   DBus paths for Victron namespace
- https://github.com/victronenergy/venus/wiki/dbus-api   DBus API from Victron
- https://www.victronenergy.com/live/ccgx:root_access   How to get root access on GX device/Venus OS
- https://shelly-api-docs.shelly.cloud/gen1/#shelly1-shelly1pm Shelly API documentation

## Discussions on the web
This module/repository has been posted on the following threads:
- https://community.victronenergy.com/questions/127339/shelly-1pm-as-pv-charger-in-venusos.html
