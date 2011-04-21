title: Manage apache2 access logs
published: 1295478000

On high traffic web servers, the log file footprint can be a serious issue when using the default system configuration. In this post, I will try to customize the way the access log are managed for respond to my own needs.

Goals
=====

My goal is to keep a proper version of the access log files each days, correctly labelled and compressed. I need to control the stock on a day basis, with a customizable retention (3 day in this case).

Default setup (Debian based system)
===================================

On Debian based system (Ubuntu 10.04 LTS), these options are log related :

```apache
...
#
# HostnameLookups: Log the names of clients or just their IP addresses
# e.g., www.apache.org (on) or 204.62.129.132 (off).
# The default is off because it'd be overall better for the net if people
# had to knowingly turn this feature on, since enabling it means that
# each client request will result in AT LEAST one lookup request to the
# nameserver.
#
HostnameLookups Off

# ErrorLog: The location of the error log file.
# If you do not specify an ErrorLog directive within a
# container, error messages relating to that virtual host will be
# logged here.  If you *do* define an error logfile for a
# container, that host's errors will be logged there and not here.
#
ErrorLog /var/log/apache2/error.log

#
# LogLevel: Control the number of messages logged to the error_log.
# Possible values include: debug, info, notice, warn, error, crit,
# alert, emerg.
#
LogLevel warn

...

#
# The following directives define some format nicknames for use with
# a CustomLog directive (see below).
# If you are behind a reverse proxy, you might want to change %h into %{X-Forwarded-For}i
#
LogFormat "%v:%p %h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" vhost_combined
LogFormat "%h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" combined
LogFormat "%h %l %u %t \"%r\" %>s %O" common
LogFormat "%{Referer}i -> %U" referer
LogFormat "%{User-agent}i" agent

#
# Define an access log for VirtualHosts that dont define their own logfile
CustomLog /var/log/apache2/other_vhosts_access.log vhost_combined
...

```

About "HostnameLookups"
-----------------------

"Log the names of clients or just their IP addresses", i prefer to work with IP addresses, and address resolutions take time, so set it off is a good choice.

more infos: [apache2 doc](http://httpd.apache.org/docs/2.0/mod/core.html#hostnamelookups)

About "ErrorLog"
----------------

This file is useful for devs (to manage php errors), and for me (to manage wrong file permissions). This is the default container, we can specify another one in the vhost files if needed.

more infos: [apache2 doc](http://httpd.apache.org/docs/2.0/mod/core.html#errorlog)

About "LogLevel"
----------------

I work on a production host so i only need the essential. I only want to log errors.

more infos: [apache2 doc](http://httpd.apache.org/docs/2.0/mod/core.html#loglevel)

LogFormat and log files
=======================

Common Log Format (CLF) contain: "%h %l %u %t \"%r\" %>s %b":

* Remote host IP (or FQDN if "HostnameLookups" specified)
* Ident (more info: [wikipedia](http://en.wikipedia.org/wiki/Ident))
* User (provided by HTTP Authentication, more info: [apache2 doc](http://httpd.apache.org/docs/2.2/howto/auth.html))
* Time when the request was received
* First line of the request (usually the HTTP type: GET, POST, etc...)
* Final status of the request (200, 404, etc...)
* Size of the response in bytes

I need to make some stats with my logfiles, so i will log more informations:

* Virtual Host + port (http or https)
* Remote Host
* Time when the request was received
* Type of the request
* Final Status of the request
* Size of the response
* Time to serve the request
* Referrer
* User Agent

Result:
    LogFormat "%v:%p %h %t  \"%r\" %>s %b %T \"%{Referer}i\" \"%{User-agent}i\"" yuweb

more infos: [apache2 doc](http://httpd.apache.org/docs/2.0/mod/mod_log_config.html)

Dummy Connection
================

> "When the Apache HTTP Server manages its child processes, it needs a way to wake up processes that are listening for new connections. To do this, it sends a simple HTTP request back to itself."

These requests appear on the log file with this message `(internal dummy connection)` and the loopback for origin (`127.0.0.1` or `::1`). I want to remove these logs from my logfile, i need to define an environment and tell to apache to not log this environment:

For IPv4:
    SetEnvIf Remote_Addr "127\.0\.0\.1" loopback

For IPv6:
    SetEnvIf Remote_Addr "\:\:1" loopback6

And i specify to not log this env in the "CustomLog":
    CustomLog "/var/log/apache2/access.log" yuweb env=!loopback6

more infos: [apache2 wiki](http://wiki.apache.org/httpd/InternalDummyConnection)

Custom setup
------------

```apache
# Logs settings
#   loglevels: debug, info, notice, warn, error, crit, alert, emerg
LogFormat "%v:%p %h %t  \"%r\" %>s %b %T \"%{Referer}i\" \"%{User-agent}i\"" yuweb
HostnameLookups Off
SetEnvIf Remote_Addr "\:\:1" loopback6
ErrorLog /var/log/apache2/error.log
LogLevel error
CustomLog "/var/log/apache2/access.log" yuweb env=!loopback6
```

Log rotation
============

Logrotate is the default log rotation utility used by debian based system. It manage apache2 logs per default.

> "logrotate  is  designed to ease administration of systems that generate large numbers of log files.  It allows automatic rotation, compression, removal, and mailing of log files.  Each log file may be handled daily, weekly, monthly, or when it grows too large."


Default configuration
---------------------

The default log rotation settings are in `/etc/logrotate.d/apache2`

```bash
/var/log/apache2/*.log {
  weekly
  missingok
  rotate 52
  compress
  delaycompress
  notifempty
  create 640 root adm
  sharedscripts
  postrotate
  if [ -f "`. /etc/apache2/envvars ; echo ${APACHE_PID_FILE:-/var/run/apache2.pid}`" ]; then
    /etc/init.d/apache2 reload > /dev/null
  fi
  endscript
}
```

In the default configuration, logs are rotated weekly, with a retention of 52 weeks, the compression is active but not on the first rotation (Because a reload of apache can keeps logging into the archival logfile for the connections non-finished).

Custom configuration
--------------------

I will set up a daily based rotation of the logfile, and keep the logs for 3 days. I want everything to be compressed. I will change the way apache2 is restarted to just tel it to write ALL the logs on the new file after the rotation is completed. I want proper name with date in the log filename.

This is my actual logrotate file for apache:

```bash
/var/log/apache2/*.log {
  daily
  missingok
  rotate 3
  compress
  create 640 root sudo
  dateext
  extension .log
  postrotate
  if [ -f "`. /etc/apache2/envvars ; echo ${APACHE_PID_FILE:-/var/run/apache2.pid}`" ]; then
    apache2ctl -k graceful > /dev/null
  fi
  endscript
}
```

more infos: [man page](http://linuxcommand.org/man_pages/logrotate8.html)

Schedule the rotation (Proper)
------------------------------

By default, logrotate is CRON sheduled (daily). It configuration file is `/etc/cron.daily/logrotate`. My problem is than i need proper schedule for hourly/daily/weekly/monthly labels in the default CRON config (`/etc/crontab`).

I need the log migrate at midnight, not 6AM, so i change the shedule like that:

```
1 *    * * *   root    cd / &amp;&amp; run-parts --report /etc/cron.hourly
5 0    * * *   root    test -x /usr/sbin/anacron || ( cd / &amp;&amp; run-parts --report /etc/cron.daily )
10 0    * * 7   root    test -x /usr/sbin/anacron || ( cd / &amp;&amp; run-parts --report /etc/cron.weekly )
15 0    1 * *   root    test -x /usr/sbin/anacron || ( cd / &amp;&amp; run-parts --report /etc/cron.monthly )
```

