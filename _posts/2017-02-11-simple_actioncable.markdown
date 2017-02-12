---
layout: post
comments: true
tags:
- "Rails 5"
- "ActionCable"
- "LimitOS"
title:  "Simple Rails 5 ActionCable Example"
---

<style type="text/css">
.custom-code { background-color: #ffe; }
</style>

Here's a <a href="https://github.com/llawlor/LimitOS/commits/25cca62eccc8b6b0318f657e7ef959ca78eec68c">very simple Rails application</a> with ActionCable working.  There is no authentication present, and no database tables required.

The application uses regular JavaScript instead of CoffeeScript, and is based off of the tutorials at <a href="http://edgeguides.rubyonrails.org/action_cable_overview.html">http://edgeguides.rubyonrails.org/action_cable_overview.html</a> and <a href="https://blog.heroku.com/real_time_rails_implementing_websockets_in_rails_5_with_action_cable">https://blog.heroku.com/real_time_rails_implementing_websockets_in_rails_5_with_action_cable</a>

In the application, clicking the "On" and "Off" buttons will update the status, and the message will be sent to any other open browser windows.

Check out all of the commits on 2017-02-11 to see the steps required to get ActionCable working.
