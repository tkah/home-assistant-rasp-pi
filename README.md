# Raspberry Pi Home Assistant Setup

These are the steps I took to set up my Home Assistant Raspberry Pi system.

__System Description__: Server Kiosk, alarm, motion sensor, door/window sensor

__Table of Contents__

* [Shopping List](#shopping-list)
* [Home Assistant, Touchscreen, Case, Pi Installation](#home-assistant-touchscreen-case-pi-installation)
* [Setup Wifi on Pi](#setup-wifi-on-pi)
* [Install Samba](#install-samba)
* [Remote Access with LetsEncrypt and DuckDNS](#remote-access-with-letsencrypt-and-duckdns)
* [Semi-Kiosk](#semi-kiosk)
* [HTML5 Push Notifications](#html5-push-notifications)
* [Z-Wave Network](#z-wave-network)
* [Sensors Introduction](#sensors-introduction)
* [Configuration.yaml](#configuration-yaml)
* [Customize.yaml](#customize-yaml)
* [Automations.yaml](#automations-yaml)
* [Groups.yaml](#groups-yaml)

## Shopping List

* [Raspberry Pi](http://a.co/8bhuT89)
* [Pi Power Supply](http://a.co/elSBidy)
* [32GB Memory Card for Pi](http://a.co/9zHgttg)
* [(Optional) Pi Heatsinks](http://a.co/1gtHfiF)
* [Touchscreen](http://a.co/aTAzU2W)
* [Touchscreen/Pi Case](http://a.co/gYAcyMi)
* [Motion Detector](http://a.co/7XCKsl4)
* [Window/Door Sensor](http://a.co/0l0PuYl)
* [Alarm](http://a.co/8DjN5IZ)
* [Z-Wave Stick](http://a.co/4sfmOS2) - Enables communication between Pi and Z-Wave components (alarm, sensors etc.)

## Home Assistant, Touchscreen, Case, Pi Installation

[Hass.io](https://home-assistant.io/hassio/) - Easiest way to install HA. However, no current support for touchscreen kiosk mode using server making an alternative necessary for now.

Use the [all-in-one installer](https://home-assistant.io/docs/installation/raspberry-pi-all-in-one/) until kiosk support in [Hass.io](https://home-assistant.io/hassio/).

1. Download Raspbian image [here](https://www.raspberrypi.org/downloads/raspbian/)
   * I chose the __Stretch Lite__ option but ended up installing a GUI anyway to make fullscreening the touch screen easier.
2. Burn image to SDCard using [Etcher](https://etcher.io/) or something similar.
3. Insert SDCard into Pi.
   * If purchased heatsinks, [attach to Pi](https://www.modmypi.com/blog/how-to-install-heat-sinks-on-the-raspberry-pi)
4. Put together Touchscreen, Case and Pi using [this video](https://youtu.be/XKVd5638T_8) as a guide.
   * Use case's ribbon, not the monitor's ribbon, to connect the touchscreen to the Pi. There is a plastic guard that slides out to allow you to easily slide in the ribbon.
   * Use the case's micro-usb cable to power both the Pi and the monitor using your single power supply.
   * If purchased heatsinks, cut out square to make room as shown at end of video.
5. Connect a keyboard to your Pi before powering on. Alternatively, you can SSH to communicate with your Pi.
6. Attach power supply.
7. Use default user: `pi` and password: `raspberry` to login to command line.
8. Install updates: `sudo apt-get update && sudo apt-get upgrade -y`
9. Localize install: `sudo raspi-config`
   * Reset default password
   * Change timezone
   * Change keyboard settings if necessary
10. `sudo reboot` to restart with new settings
11. Follow [these all-in-one instructions](https://home-assistant.io/docs/installation/raspberry-pi-all-in-one/) to complete installation of Home Assistant.
    * Be sure to add `ssh` folder as instructed
    * If installation doesn't work initially, try:
      * `sudo rm -rf fabric-home-assistant`
      * `sudo apt-get install libffi-dev libssl-dev`
      * `sudo pip install cryptography --force-reinstall`
      * `curl -O https://raw.githubusercontent.com/home-assistant/fabric-home-assistant/master/hass_rpi_installer.sh && sudo chown pi:pi hass_rpi_installer.sh && bash hass_rpi_installer.sh` or whatever command is listed in the "all-in-one" instructions above.
12. It might not be a bad idea, once all of the setup is complete, to add a login password to Home Assistant. The instructions can be found [here](https://home-assistant.io/docs/configuration/basic/).
13. Install a touch keyboard with: `sudo apt-get install matchbox-keyboard`

## Setup Wifi on Pi

Follow [these instructions](https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md) to setup wifi on your pi.

## Install Samba

To make your Pi a shared drive you can access from your local network, use [Samba](https://www.samba.org/). Installation instructions [here](https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md).

## Remote Access with LetsEncrypt and DuckDNS

To access your router remotely, such as from your smartphone while away on vacation, you might consider LetsEncrypt and DuckDNS. An overview of what these are and how to install them can be found [here]( https://home-assistant.io/docs/ecosystem/certificates/lets_encrypt/).

I had some issues with my router not seemingly allowing me to set local ports as indicated in those instructions. So, I also used some of the advice [in this forum post](https://community.home-assistant.io/t/guide-how-to-set-up-duckdns-ssl-and-chrome-push-notifications/9722). Specifically, I used the permissions, and forwarding rule from the second link, but only after I had already installed LetsEncrypt. So, I'd recommend using the first link, as it's much more descriptive, and only coming to the second if you get stuck.

Additionally, I had set my the `base_url` in my `configuration.yaml` without the port. But after following the instructions in the second link about port 8123, I had to add that to the end of my `base_url`: XXX.XXX.XX.XX:8123. I had to use the internal IP because my router doesn't support `loopback`ing as described below and referenced in the link at the end of the first paragraph of this section.

## Semi-Kiosk

I had a lot of problems finding a way to make a Chromium kiosk with my server that would give me access to a virtual keyboard. Instead, I upgraded my originally Lite install to use a GUI (PIXEL). I use an automated script to start directly into Chromium with flags to ignore faulty certificates as accessing my Pi via its internal IP address has caused a great deal of problems and my router won't allow `loopback`ing, which is why I'm using the internal IP address. No `loopback`ing also denies my setup the option of using text-to-speech (tts) between HA and my Google Home.

1. `sudo apt-get install --no-install-recommends xserver-xorg xinit lightdm`
   * The `x` stuff handles screen sessions. `lightdm` allows automatic logging in.
2. `sudo install matchbox-keyboard`
   * `matchbox-keyboard` is a virtual keyboard that integrated well into the GUI
   * `florence` is an alternative, though I didn't have much luck with it
3. `sudo apt-get install vim`
   * `vim` is a popular text editor
4. `sudo vim /etc/lightdm/lightdm.conf`
   * Uncomment and change the following lines in the `[Seat:*]` section:
     * `autologin-user=pi`
     * `autologin-user-timeout=0`
   * This allows your _pi_ user to automatically login.
   * Many above steps taken from [here](https://tamarisk.it/raspberry-pi-kiosk-mode-using-raspbian-lite/).
5. `vim ~/.config/lxsession/LXDE-pi/autostart`
   * Delete or comment the line with `@xscreensaver -no-splash`
   * Add the following at the end of the file:
     * `@unclutter`
     * `@chromium-browser --ignore-certificate-errors --test-type https://XXX.XXX.XX.XX:8123` <- Your pi's address (internal IP if similarly dealing with no `loopback` or your DuckDNS is `loopback`ing is supported)
   * Taken from [here](https://www.raspberrypi.org/forums/viewtopic.php?f=91&t=163316).
6. After running `sudo reboot`, you'll eventually be brought to the GUI with a Chromium window containing your HA instance. If this isn't loading, you should be able to hit __reload__ to get it to load correctly.
   * Entering the password can be tricky unless you want to continually be connecting a keyboard. That's why you installed the virtual keyboard, which is still a bit of a pain. Tap the _Raspberry Pi_ icon in the top-left corner. Under __Accessibility__ you should see a _keyboard_ option. Open the keyboard from here and enter your password.
   * Once you've logged-in to HA, go to Chrome settings (3 vertical dots) and tap the icon all the way to the right of the zoom-in and zoom-out icons to enter `fullscreen` mode with HA. You can exit fullscreen with a two-finger tap and selecting the appropriate option.

## HTML5 Push Notifications

To receive HTML push notifications on my mobile device, I followed the instructions outlined in [this forum post](https://community.home-assistant.io/t/guide-how-to-set-up-duckdns-ssl-and-chrome-push-notifications/9722) from above. Some things had to change, such as the location for the `index.html` file where the `meta` tag goes. I have `python3.5` installed so the directory was slightly different. Also, the process of registering my domain with Google was slightly different than what was described. I registered through Google's Search Console. Lastly, the service in HA is now called `notify` rather than `html5`. Still, most of it was spot on. However, I don't know that installing `pywebpush` was necessary as it may be included in the all-in-one installer. Not sure about that, though.

## Z-Wave Network

Each item on the shopping list above has an instruction pamphlet describing how to sync it with the Z-Wave stick. Essentially, you just give the device power, by removing the battery tab or plugging it in, and hit the button on the stick. Plug the stick into your Pi, reboot and you should have a new card listing all of your devices. If you go to `Confiuration` > `Z-Wave` > `Node Management`, you'll find a dropdown list of all your devices. You can customize their names through this list, which will also update their `entity_id`s.

[More Info](https://home-assistant.io/blog/2017/06/15/zwave-entity-ids/)

## Alarm Panel

One of the main reasons I decided to use HA was to create a security alarm. There are a few options for this. The one I decided on was the [Manual Alarm Control Panel](https://home-assistant.io/components/alarm_control_panel.manual/). I made two - one to fire a silent alarm notification and the other to set off my alarm siren.

```alarm_control_panel:
  - platform: manual
    name: "Silent Alarm"
    pending_time: 45
    trigger_time: 5
    disarm_after_trigger: false
  - platform: manual
    name: "Noisy Alarm"
    pending_time: 45
    trigger_time: 5
    disarm_after_trigger: false
```

## Sensors Introduction

The icons at the top of your Home Assistant UI are sensors. They give you quick information about the state of various things you are tracking. The above forum posting has instructions on how to add a sensor to tell you for how long your TLS certificate is valid. Another popular sensor one is for weather. [Here are instructions](https://home-assistant.io/components/sensor.darksky/) for adding a different weather sensor via Dark Sky. You can also hide sensors, including those that are automatically generated for you. [This document](https://home-assistant.io/docs/configuration/customizing-devices/) outlines the steps necessary for that. You can find the `entity_id` for your sensors by clicking on them.

## Configuration.yaml

This is what my `configuration.yaml` currently looks like. The more difficult thing to understand initially was how `scripts` and `automations` relate. What I've found is that `scripts` are good ways to arrange a series of events, which you can call from an automation. Automations are ways to trigger certain actions on a state change - motion detected, window opened - and to place conditions on those triggers - alarm active.

```homeassistant:
  # Name of the location where Home Assistant is running
  name: Home
  # Location required to calculate the time the sun rises and sets
  latitude:
  longitude:
  # Impacts weather/sunrise data (altitude above sea level in meters)
  elevation: 1596
  # metric for Metric, imperial for Imperial
  unit_system: imperial
  # Pick yours from here: http://en.wikipedia.org/wiki/List_of_tz_database_time_zones
  time_zone: America/Denver
  customize: !include customize.yaml

#android_ip_webcam:
#  - host: 192.168.0.20
#    username:
#    password:
#    sensors:
#      - battery_level
#      - battery_temp
#      - motion
#    switches:
#      - video_recording
#      - night_vision

alarm_control_panel:
  - platform: manual
    name: "Silent Alarm"
    pending_time: 45
    trigger_time: 5
    disarm_after_trigger: false
  - platform: manual
    name: "Noisy Alarm"
    pending_time: 45
    trigger_time: 5
    disarm_after_trigger: false

# Enables the frontend
frontend:

# Enables configuration UI
config:

http:
  # Uncomment this to add a password (recommended!)
  api_password:
  ssl_certificate:
  ssl_key:
  base_url:

# Checks for available updates
# Note: This component will send some information about your system to
# the developers to assist with development of Home Assistant.
# For more information, please see:
# https://home-assistant.io/blog/2016/10/25/explaining-the-updater/

updater:
  # Optional, allows Home Assistant developers to focus on popular components.
  # include_used_components: true

notify:
  platform: html5
  gcm_api_key:
  gcm_sender_id:

# Discover some devices automatically
discovery:

# Allows you to issue voice commands from the frontend in enabled browsers
conversation:

# Enables support for tracking state changes over time
history:

# View all events in a logbook
logbook:

# Track the sun
sun:

sensor:
  - platform: darksky
    api_key:
    forecast: 1
    monitored_conditions:
      - summary
      - precip_probability
      - temperature_max
      - temperature_min
  - platform: command_line
    name: SSL Cert Expiry
    unit_of_measurement: days
    scan_interval: 10800
    command: "ssl-cert-check ..."

script:
  silent_alarm_script:
    sequence:
      - alias: Push Notification Silent Alarm
        service: notify.notify
        data:
          message: 'Alert! Alarm has gone off.'
  noisy_alarm_script:
    sequence:
      - alias: Siren Noisy Alarm
        service: switch.turn_on
        entity_id: switch.gocontrol_siren_and_strobe_switch

# Text to speech
tts:
  - platform: google

zwave:
  usb_path: /dev/ttyACM0

group: !include groups.yaml
automation: !include automations.yaml
```

## Customize.yaml

I got a bit tired of turning off all the sensors associated with my ZWave components, so I put them in a separate file.

You cna find the names of the sensors in the UI by clicking on the sensor you'd like to hide.

```sensor.living_room_motion_detector_alarm_level:
  hidden: true
sensor.living_room_motion_detector_alarm_type:
  hidden: true
sensor.living_room_motion_detector_burglar:
  hidden: true
sensor.living_room_motion_detector_power_management:
  hidden: true
sensor.living_room_motion_detector_sourcenodeid:
  hidden: true
sensor.office_window_access_control:
  hidden: true
sensor.office_window_alarm_level:
  hidden: true
sensor.office_window_alarm_type:
  hidden: true
sensor.office_window_burglar:
  hidden: true
sensor.office_window_power_management:
  hidden: true
sensor.office_window_sourcenodeid:
  hidden: true
sensor.sliding_glass_door_access_control:
  hidden: true
sensor.sliding_glass_door_alarm_level:
  hidden: true
sensor.sliding_glass_door_alarm_type:
  hidden: true
sensor.sliding_glass_door_burglar:
  hidden: true
sensor.sliding_glass_door_power_management:
  hidden: true
sensor.sliding_glass_door_sourcenodeid:
  hidden: true
sensor.lyras_window_access_control:
  hidden: true
sensor.lyras_window_alarm_level:
  hidden: true
sensor.lyras_window_alarm_type:
  hidden: true
sensor.lyras_window_burglar:
  hidden: true
sensor.lyras_window_power_management:
  hidden: true
sensor.lyras_window_sourcenodeid:
  hidden: true
script.silent_alarm_script:
  hidden: true
script.noisy_alarm_script:
  hidden: true
automation.arm_silent_alarm:
  hidden: true
automation.arm_noisy_alarm:
  hidden: true
```

## Automations.yaml

I made a couple automations to trigger an alarm if any sensors in my ZWave group go off. I also have some for turning on and off the night vision on my phone camera depending on the sun's position.

The entity names are essentially the lower-case type of entity followed by a period followed by the lower-case name with underscores in place of spaces.

```- alias: Arm Silent Alarm
  trigger:
    - platform: state
      entity_id: group.alarm_triggers
      from: 'off'
      to: 'on'
  condition:
    condition: state
    entity_id: alarm_control_panel.silent_alarm
    state: armed_away
  action:
    service: script.turn_on
    entity_id: script.silent_alarm_script

- alias: Arm Noisy Alarm
  trigger:
    - platform: state
      entity_id: group.alarm_triggers
      from: 'off'
      to: 'on'
  condition:
    condition: state
    entity_id: alarm_control_panel.noisy_alarm
    state: armed_away
  action:
    - service: script.turn_on
      entity_id: script.noisy_alarm_script
    - service: script.turn_on
      entity_id: script.silent_alarm_script

- alias: Night Vision Camera On
  trigger:
    - platform: sun
      event: sunset
  action:
    - service: switch.turn_on
      entity_id: switch.ip_webcam_night_vision

- alias: Night Vision Camera Off
  trigger:
    - platform: sun
      event: sunrise
  action:
    - service: switch.turn_off
      entity_id: switch.ip_webcam_night_vision
```

## Groups.yaml

I made a ZWave group to watch in order to trigger my automations.

```alarm_triggers:
    name: Alarm Triggers
    entities:
      - binary_sensor.living_room_motion_detector_sensor
      - binary_sensor.lyras_window_sensor
      - binary_sensor.office_window_sensor
      - binary_sensor.sliding_glass_door_sensor
```

## Useful Commands

* Restart Samba: `sudo service smbd restart`
