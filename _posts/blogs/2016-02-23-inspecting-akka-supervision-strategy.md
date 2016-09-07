---
layout: post
title:  Inspecting Akka's Supervision Strategy
link: http://www.cakesolutions.net/teamblogs/inspecting-akkas-supervision-strategy
categories: blogs
---
Akka's OneForOneStrategy has a maxNrOfRetries parameter, that would result in a Stop rather than a Restart if its limit was exceeded. How can we Escalate instead?
