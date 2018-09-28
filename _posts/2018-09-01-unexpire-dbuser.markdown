---
layout: post
title:  "Oracle unexpire user password"
date:   2018-09-01 16:33:34 -0700
categories: database
---
Recently, I came across a situation where the database user account had expired. This user account was used by docker instance and was more complicated to update the password in that instance. So I tried to find ways to keep the same password but make the account unexpired. I found this [link](https://www.krenger.ch/blog/oracle-workaround-for-password-unexpire/) which I thought was interesting and saved my day.