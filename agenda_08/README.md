# Splunkd Sticky Sessions

## Problem:

As most of you probbaly know, just like `splunkweb`, which is used to access the Web-UI, `splunkd` which is usually listening on port 8089 can also be used to run searches programatically. In the case of a SHC, you may have a Load Balancer in front of the SHC on port 8089, so you can have those scripts/programs access the SHC on that port in a highly avaialable fashion.

When using this setup, you may encounter authentication issues with `[HTTP 401] Client is not authenticated` message appearing in the log files.

## Solution:

A lot of people remember to set stickiness for the web UI on the load-balancer, but forget that `splunkd` is also a web server. What happens is that the request may end up on another server, 

In order to use cookie authentication, you first need to add `allowCookieAuth = true` in `server.conf` in the `[httpServer]` stanza. 

The default is `true`, but you need to add `cookie=1` in the POST when you hit the `auth/login` endpoint. If a cookie is used, you should see the `Set-Cookie` in the response header.
