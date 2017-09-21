# Raspberry Pi Home Assistant Setup

These are the steps I took to set up my Home Assistant Raspberry Pi system. I've written them mostly so I can recreate the process again without having to search everywhere for the details. However, they should be useful to others setting up a similar system. The only knowledge I assume you have is some basic familiarity with the command line and how to save/close files using `nano` or `vim`.

__System Description__: Pi, server kiosk, alarm, motion sensor, door/window sensor, old Android smartphone

__Table of Contents__

* [Shopping List](#shopping-list)
* [Home Assistant, Touchscreen, Case, Pi Installation](#home-assistant-touchscreen-case-pi-installation)
* [Setup Wifi on Pi](#setup-wifi-on-pi)
* [Install Samba](#install-samba)
* [Auto-Updates](#auto-updates)
* [Remote Access with Nginx, LetsEncrypt, and DuckDNS](#remote-access-with-nginx-letsencrypt-and-duckdns)
* [Semi-Kiosk](#semi-kiosk)
* [HTML5 Push Notifications](#html5-push-notifications)
* [Z-Wave Network](#z-wave-network)
* [Sensors Introduction](#sensors-introduction)
* [Android IP Webcam](#android-ip-webcam)
* [Configuration yaml](#configuration-yaml)
* [Customize yaml](#customize-yaml)
* [Automations yaml](#automations-yaml)
* [Scripts yaml](#scripts-yaml)
* [Groups yaml](#groups-yaml)

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

[Hass.io](https://home-assistant.io/hassio/) - Easiest way to install HA. However, no current support for touchscreen kiosk mode using server.

Another alternative is the [all-in-one installer](https://home-assistant.io/docs/installation/raspberry-pi-all-in-one/). The issue I had with this was that the installer was behind the newest version of Raspbian when I tried it.

So, I installed using a [python virtual environment](https://home-assistant.io/docs/installation/virtualenv/). This might be the more complex installer for non-programmers, but it's really not so bad.

1. Download Raspbian image [here](https://www.raspberrypi.org/downloads/raspbian/)
   * I chose the __Stretch Desktop__ to make setting up a Kiosk mode easier.
2. Burn image to SDCard using [Etcher](https://etcher.io/) or something similar.
3. If you're going to `ssh` into your pi, then you'll want to add an `ssh` directory to the root of your image.
4. Insert SDCard into Pi.
   * If purchased heatsinks, [attach to Pi](https://www.modmypi.com/blog/how-to-install-heat-sinks-on-the-raspberry-pi)
5. Put together Touchscreen, Case and Pi using [this video](https://youtu.be/XKVd5638T_8) as a guide.
   * Use case's ribbon, not the monitor's ribbon, to connect the touchscreen to the Pi. There is a plastic guard that slides out to allow you to easily slide in the ribbon.
   * Use the case's micro-usb cable to power both the Pi and the monitor using your single power supply.
   * If purchased heatsinks, cut out square to make room as shown at end of video.
6. Connect a keyboard to your Pi before powering on. Alternatively, you can SSH to communicate with your Pi.
7. Attach power supply.
8. Use default user: `pi` and password: `raspberry` to login to command line.
9. Install updates: `sudo apt-get update && sudo apt-get upgrade -y`
10. Localize install: `sudo raspi-config`
    * Reset default password
    * Change timezone
    * Change keyboard settings if necessary
11. `sudo reboot` to restart with new settings
12. Follow [virtualenv instructions](https://home-assistant.io/docs/installation/virtualenv/) to complete installation of Home Assistant.
    * This site also has simple steps for upgrading your HA install.
13. Follow [these instructions](https://home-assistant.io/docs/autostart/systemd/) to set your HA instance to automatically start when your pi starts.
14. This might be a good time to go ahead and run `sudo apt-get install libudev-dev`, which will allow you to run ZWave components.
15. Once all of the setup is complete, add a login password to Home Assistant. The instructions can be found [here](https://home-assistant.io/docs/configuration/basic/).
16. Set a static ip address as described in step 1 [here](https://home-assistant.io/docs/ecosystem/certificates/lets_encrypt/#1---set-your-device-to-have-a-static-ip-address).

## Setup Wifi on Pi

Follow [these instructions](https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md) to setup wifi on your pi.

## Install Samba

To make your Pi a shared drive you can access from your local network, use [Samba](https://www.samba.org/). Installation instructions [here](https://www.youtube.com/watch?v=iQwWEsuRWUw).

## Auto-Updates

1. `sudo apt-get install unattended-upgrades`
   * Accept defaults
2. `sudo apt-get install update-notifier-common`
   * Will automatically restart your system when needed for updates to install
   * `sudo vim /etc/apt/apt.conf.d/50unattended-upgrades` - Uncomment and change to **true** the line concerning automatic restart

## Remote Access with Nginx, LetsEncrypt, and DuckDNS

To access your router remotely, such as from your smartphone while away on vacation, you might consider LetsEncrypt and DuckDNS. An overview of what these are can be found [here]( https://home-assistant.io/docs/ecosystem/certificates/lets_encrypt/#0---gain-a-basic-level-of-understanding-around-ip-addresses-port-numbers-and-port-forwarding).

My router is somewhat limited in the realm of port forwarding. Using `nginx` as a proxy allowed me to side-step a couple of the limitations stemming from my router. It still won't let me connect directly to my duckdns.org domain from within my house, but it does resolve some of the issues I had without it concerning the use of Google TTS.

1. Install `nginx`: `sudo apt-get install nginx`
2. Stop nginx: `sudo /etc/init.d/nginx stop`
3. Follow the steps laid out [here](https://home-assistant.io/docs/ecosystem/nginx/). 
   * Get a duckdns.org domain following step 3 [here](https://home-assistant.io/docs/ecosystem/certificates/lets_encrypt/#3---set-up-a-duckdns-account).
   * Get a LetsEncrypt cert following step 4 [here](https://home-assistant.io/docs/ecosystem/certificates/lets_encrypt/#4---obtain-a-tlsssl-certificate-from-lets-encrypt).
   * Use your internal HA ip address as `proxy_pass` rather than `localhost` as listed in the script there.
   * You'll need to change the owner of the `/etc/letsencrypt` directory, or the permissions.
   * The rest of the instructions can be followed as is.
   * Bonus: [Create a sensor to tell you how much time is left on your TLS cert](https://home-assistant.io/docs/ecosystem/certificates/lets_encrypt/#7---set-up-a-sensor-to-monitor-the-expiry-date-of-the-certificate).

~~I had some issues with my router not seemingly allowing me to set local ports as indicated in those instructions. So, I also used some of the advice [in this forum post](https://community.home-assistant.io/t/guide-how-to-set-up-duckdns-ssl-and-chrome-push-notifications/9722). Specifically, I used the permissions, and forwarding rule from the second link, but only after I had already installed LetsEncrypt. So, I'd recommend using the first link, as it's much more descriptive, and only coming to the second if you get stuck.~~

~~Additionally, I had set my the `base_url` in my `configuration.yaml` without the port. But after following the instructions in the second link about port 8123, I had to add that to the end of my `base_url`: XXX.XXX.XX.XX:8123. I had to use the internal IP because my router doesn't support `loopback`ing as described below and referenced in the link at the end of the first paragraph of this section.~~

## Semi-Kiosk

1. `sudo install florence`
   * `florence` is a virtual keyboard that's integrated well into the GUI
2. `sudo apt-get install at-spi2-core`
   * `florence` needs this to work properly
3. `sudo apt-get install vim`
   * `vim` is a popular text editor
4. `vim ~/.config/lxsession/LXDE-pi/autostart`
   * Delete or comment the line with `@xscreensaver -no-splash`
   * Add the following at the end of the file:
     * `@chromium-browser --ignore-certificate-errors --test-type https://XXX.XXX.XX.XX` <- Your pi's address (internal IP if no `loopback`ing on your router or your DuckDNS if `loopback`ing is supported)
   * Taken from [here](https://www.raspberrypi.org/forums/viewtopic.php?f=91&t=163316).
5. After running `sudo reboot`, you'll eventually be brought to the GUI with a Chromium window containing your HA instance. If this isn't loading, you should be able to hit __reload__ to get it to load correctly.
   * Entering the password can be tricky unless you want to continually be connecting a keyboard. That's why you installed the virtual keyboard, which is still a bit of a pain. Tap the _Raspberry Pi_ icon in the top-left corner. Under __Accessibility__ you should see a _keyboard_ option. Open the keyboard from here and use it to enter your password.
   * Once you've logged-in to HA, go to Chrome settings (3 vertical dots) and tap the icon all the way to the right of the zoom-in and zoom-out icons to enter `fullscreen` mode with HA. You can exit fullscreen with a two-finger tap and selecting the appropriate option.

## HTML5 Push Notifications

To receive HTML push notifications on my mobile device, I followed the instructions outlined in [this forum post](https://community.home-assistant.io/t/guide-how-to-set-up-duckdns-ssl-and-chrome-push-notifications/9722) from above. Some things had to change, such as the location for the `index.html` file where the `meta` tag goes. I have `python3.5` installed so the directory was slightly different. Also, the process of registering my domain with Google was slightly different than what was described. I registered through the [Google Search Console](https://www.google.com/webmasters/tools/). Lastly, the service in HA is now called `notify` rather than `html5`. Removing and reinstalling `pywebpush` was unnecessary as well. Still, most of it was spot on.

## Z-Wave Network

Each item on the shopping list above has an instruction pamphlet describing how to sync it with the Z-Wave stick. Essentially, you just give the device power, by removing the battery tab or plugging it in, and hit the button on the stick right after. Plug the stick into your Pi, reboot and you should have a new card listing all of your devices (it may take some time for it to appear with the correct information). If you go to `Confiuration` > `Z-Wave` > `Node Management`, you'll find a dropdown list of all your devices. You can customize their names through this list, which will also update their `entity_id`s.

[More Info](https://home-assistant.io/blog/2017/06/15/zwave-entity-ids/)

It's also nice to set a symlink to your USB device so that if you move your dongle or add other peripherals, you can always tell HA where your ZWave stick is. These instructions come from step 4 [here](https://jrowberg.io/2017/05/29/home-automation-guide-home-assistant-siri-alexa-z-wave-icloud/).

1. `sudo nano /etc/udev/rules.d/99-usb-serial.rules`
2. Add this line: `SUBSYSTEM=="tty", ACTION=="add", ATTRS{idVendor}=="0658", ATTRS{idProduct}=="0200", SYMLINK+="zwave"`
3. Restart udev: `sudo systemctl restart udev.service`
4. Add the following to your `configuration.yaml`:
   ```yaml
      zwave:
        usb_path: /dev/zwave
   ```

## Alarm Panel

One of the main reasons I decided to use HA was to create a security alarm. There are a few options for this. The one I decided on was the [Manual Alarm Control Panel](https://home-assistant.io/components/alarm_control_panel.manual/). I made two - one to fire a silent alarm notification and the other to set off my alarm siren.

The status of these will be used by my **automations** to determine whether or not to fire the alarm script(s).

```yaml
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
```

## Sensors Introduction

The icons at the top of your Home Assistant UI are sensors. They give you quick information about the state of various things you are tracking. For example, a popular sensor for weather is **DarkSky**. [Here are instructions](https://home-assistant.io/components/sensor.darksky/) for adding it to your setup. You can also hide sensors, including those that are automatically generated for you. [This document](https://home-assistant.io/docs/configuration/customizing-devices/) outlines the steps necessary for that. You can find the `entity_id` for your sensors by clicking on them.

## Android IP Webcam

I do plan on buying a nice webcam at some point, but for now I'm using an old Samsung S3 as my front-door camera. You can turn your Android smartphone into a webcam by downloading the [IP Webcam](https://play.google.com/store/apps/details?id=com.pas.webcam.pro&hl=en) app from the Google Play store. There are free and paid versions. Once you install the app, you can configure a few things in the settings.

* **Motion and sound detection**
   * Check *Enable motion detection*
   * Check *Enable sound detection*
* **Data logging**
   * Check *Enable data logging*
   * This setting and those previous will allow you to use your phone in HA for motion detection.
* Check **Stream device on boot** if your phone is a dedicated camera.
* If you want to email pictures to yourself when motion is detected:
   * **Plug-ins** > *Install and manage scripts* > *Email on modet*
   * When that plugin downloads, go back a screen and turn it on.
   * After tapping the on button, tap on *Email on modet* in the **Plug-ins** page to take you to an options page for the plugin.
   * Tap on *Install uploader plugin* to be taken back to the Play store to another app by the same developer which will enable you to send yourself emails.
   * Once this uploader plugin is installed you'll need to tap on *Select uploader* and choose *Email uploader* which will take you to another options page where you can fill in your email credentials.
   * If you're using Gmail along with 2-factor authentication, you can get an "App Password" by following [these instructions](https://support.google.com/accounts/answer/185833?hl=en).
   * More app config info can be found about half-way down [this *howtogeek* tutorial](https://www.howtogeek.com/139373/how-to-turn-an-old-android-phone-into-a-networked-security-camera/).

You'll also want to give your phone a static IP address in order to tie things in on the HA side. For more on configuring HA see [here](https://www.howtogeek.com/139373/how-to-turn-an-old-android-phone-into-a-networked-security-camera/).

```yaml
android_ip_webcam:
  - host: !secret front_door_webcam_ip_address
    name: 'Front Door'
    username: !secret front_door_webcam_user
    password: !secret front_door_webcam_password
    sensors:
      - battery_level
      - battery_temp
      - motion
    switches:
      - video_recording
      - night_vision
```

## Secrets

If you'd like to share your configuration files with other people without worrying about exposing your passwords, IP addresses, etc., you can use [secrets](https://home-assistant.io/docs/configuration/secrets/).

## Configuration yaml

This is what my `configuration.yaml` currently looks like. The more difficult thing to understand initially was how `scripts` and `automations` relate. What I've found is that `scripts` are good ways to arrange a series of events, which you can call from any number of automations. Automations are ways to trigger certain actions on a state change - motion detected, window opened - and to place conditions on those triggers - alarm active.

```yaml
homeassistant:
  # Name of the location where Home Assistant is running
  name: Home
  # Location required to calculate the time the sun rises and sets
  latitude: !secret ha_latitude
  longitude: !secret ha_longitude
  # Impacts weather/sunrise data (altitude above sea level in meters)
  elevation:
  # metric for Metric, imperial for Imperial
  unit_system: imperial
  # Pick yours from here: http://en.wikipedia.org/wiki/List_of_tz_database_time_zones
  time_zone: America/Denver
  customize: !include customize.yaml

android_ip_webcam:
  - host: !secret front_door_webcam_ip_address
    name: 'Front Door'
    username: !secret front_door_webcam_user
    password: !secret front_door_webcam_password
    sensors:
      - battery_level
      - battery_temp
      - motion
    switches:
      - video_recording
      - night_vision

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
  api_password: !secret http_password

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
  gcm_api_key: !secret google_notify_gcm_api_key
  gcm_sender_id: !secret google_notify_gcm_sender_id

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
  - platform: command_line
    name: SSL Cert Expiry
    unit_of_measurement: days
    scan_interval: 10800
    command: !secret expiration_sensor_command

script: !include scripts.yaml

# Text to speech
tts:
  - platform: google

zwave:
  usb_path: /dev/zwave

group: !include groups.yaml
automation: !include automations.yaml
```

## Customize yaml

My ZWave devices added a bunch of sensors by default. I wanted to hide them, but didn't want all of those extra lines in my main configuration file. So, I put them in a separate `customize.yaml` file.

You can find the names of the sensors in the UI by clicking on the sensor you'd like to hide.

```yaml
sensor.living_room_motion_detector_alarm_level:
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
script.silent_alarm_script:
  hidden: true
script.noisy_alarm_script:
  hidden: true
script.front_door_motion:
  hidden: true
script.night_vision_off:
  hidden: true
script.night_vision_on:
  hidden: true
automation.arm_silent_alarm:
  hidden: true
automation.arm_noisy_alarm:
  hidden: true
automation.front_door_motion:
  hidden: true
automation.night_vision_camera_off:
  hidden: true
automation.night_vision_camera_on:
  hidden: true
```

## Automations yaml

I made a couple automations to trigger an alarm if any sensors in my ZWave group go off. I also have some for turning on and off the night vision on my phone camera depending on the sun's position.

The entity names are essentially the lower-case type of entity followed by a period followed by the lower-case name with underscores in place of spaces.

```yaml
- alias: Arm Silent Alarm
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

- alias: Front Door Motion
  trigger:
    - platform: numeric_state
      entity_id: sensor.front_door_motion
      above: '5'
  action:
    - service: script.turn_on
      entity_id: script.front_door_motion

- alias: Night Vision Camera On
  trigger:
    - platform: sun
      event: sunset
  action:
    - service: script.turn_on
      entity_id: script.night_vision_on

- alias: Night Vision Camera Off
  trigger:
    - platform: sun
      event: sunrise
  action:
    - service: script.turn_on
      entity_id: script.night_vision_off
```

## Scripts yaml

```yaml
silent_alarm_script:
  sequence:
    - alias: Push Notification Silent Alarm
      service: notify.notify
      data:
        message: 'Alert! Alarm has gone off.'
    - alias: Google Home Warning
      service: tts.google_say
      entity_id: media_player.living_room_home
      data:
        message: 'Intruder detected. The authorities have been notified.'
noisy_alarm_script:
  sequence:
    - alias: Siren Noisy Alarm
      service: switch.turn_on
      entity_id: switch.siren
front_door_motion:
  sequence:
    - alias: Google Home Front Door
      service: tts.google_say
      entity_id: media_player.living_room_home
      data:
        message: 'Someone is near the front door.'
night_vision_on:
  sequence:
    - alias: Night Vision On
      service: switch.turn_on
      entity_id: switch.front_door_night_vision
night_vision_off:
  sequence:
    - alias: Night Vision Off
      service: switch.turn_off
      entity_id: switch.front_door_night_vision
```

## Groups yaml

I made a ZWave group to watch in order to trigger my automations.

```yaml
alarm_triggers:
    name: Alarm Triggers
    entities:
      - binary_sensor.living_room_motion_detector_sensor
      - binary_sensor.lyras_window_sensor
      - binary_sensor.office_window_sensor
      - binary_sensor.sliding_glass_door_sensor
```

## Useful Commands

* Restart Samba: `sudo service smbd restart`
* HA Status: `sudo systemctl status home-assistant@homeassistant.service -l`
* Restart HA: `sudo systemctl restart home-assistant@homeassistant.service -l`
* Most recently installed packages: `grep " install " /var/log/dpkg.log`
* Restart `nginx`: `sudo /etc/init.d/nginx restart`
