---
layout: post
comments: true
tags:
- "Raspberry Pi"
title:  "Setting up a Raspberry Pi without a monitor or keyboard"
---

<style type="text/css">
.custom-code { background-color: #ffe; }
</style>

<ol>
  <li><a href="https://www.raspberrypi.com/software/">Install Raspberry Pi OS onto the Raspberry Pi via an SD card</a></li>
  <li>Add a blank file named <code>ssh</code> (without an extension) to the boot partition of the Raspberry Pi</li>
  <li>
    Add a file called "wpa_supplicant.conf" to the boot partition, with the following contents:
    <br>
<pre>
country=US
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    ssid="YOUR_WIFI_NAME"
    psk="YOUR_WIFI_PASSWORD"
}</pre>
  </li>
  <li>
    Connect to the Raspberry Pi via SSH, for example: <code>ssh pi@raspberrypi.local</code>.  The default username is <em>pi</em>, and the default password is <em>raspberry</em>.
  </li>
  <li>Update the repositories via <code>sudo apt-get update</code></li>
  <li>Install RDP on the Raspberry Pi via <code>sudo apt-get install xrdp</code></li>
  <li>Create a new user that you'll use to connect, via <code>sudo adduser NEW_USER_NAME</code></li>
  <li>From your computer, connect to the Raspberry Pi via Remote Desktop Protocol (<em>raspberrypi</em> should work as the computer name to connect to).  Use the username and password that you created in the previous step, since the <em>pi</em> user by default can't use Remote Desktop Protocol.</li>
</ol>
