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
  <li><a href="https://www.raspberrypi.org/downloads/raspbian/">Install Raspbian onto the Raspberry Pi via an SD card</a></li>
  <li>Add a blank file named <code>ssh</code> (without an extension) to the boot partition of the Raspberry Pi</li>
  <li>Connect the Raspberry Pi to your router via an ethernet cable, and power up the Raspberry Pi</li>
  <li>Find the Raspberry Pi's IP address via the router's admin interface</li>
  <li>
    Connect to the Raspberry Pi via SSH, for example: <code>ssh pi@192.168.1.187</code> , making sure to use your Raspberry Pi's IP address.  The default username is <em>pi</em>, and the default password is <em>raspberry</em>.
  </li>
  <li>Update the repositories via <code>sudo apt-get update</code></li>
  <li>Install RDP on the Raspberry Pi via <code>sudo apt-get install xrdp</code></li>
  <li>From your computer, connect to the Raspberry Pi via Remote Desktop Protocol.  Use the same IP address, username, and password as before.</li>
  <li>Using the RDP connection, configure Wi-Fi on the Raspberry Pi</li>
  <li>Remove the ethernet cable</li>
  <li>Find the Raspberry Pi's wireless IP address via your router's admin interface, for example 192.168.1.188</li>
  <li>Connect to the Raspberry Pi via RDP (using the wireless IP address you just found, for example 192.168.1.188)</li>
</ol>
