---
layout: post
comments: true
tags:
- "Raspberry Pi"
title:  "Setting up I2C on a Raspberry Pi"
---

<style type="text/css">
.custom-code { background-color: #ffe; }
</style>

Install the I2C tools on your Raspberry Pi: <code>sudo apt-get install i2c-tools</code>

Enable the I2C by editing the <em>/boot/config.txt</em> file:
<code>sudo nano /boot/config.txt</code>

Add the following two lines:

<code>dtparam=i2c1=on</code>
<br>
<code>dtparam=i2c_arm=on</code>

Reboot the Raspberry Pi:
<code>sudo reboot</code>

Test that I2C is set up:
<code>sudo i2cdetect -y 1</code>
