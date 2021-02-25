# WordPress ModSecurity Rule Set (WPRS)
This ruleset extends the OWASP CRS (https://github.com/SpiderLabs/owasp-modsecurity-crs) and includes specific rules in order to protect the most critical aspects of the famous WordPress CMS.

## Table of contents
- [Install](https://github.com/theMiddleBlue/wordpress-modsecurity-ruleset#install)
  - [Use with OWASP CRS](https://github.com/theMiddleBlue/wordpress-modsecurity-ruleset#use-with-owasp-crs)
- [Configuration](https://github.com/theMiddleBlue/wordpress-modsecurity-ruleset#configurations)
  - [Real Client IP Address](https://github.com/theMiddleBlue/wordpress-modsecurity-ruleset#real-client-ip-address)
  - [Mitigate Brute-force](https://github.com/theMiddleBlue/wordpress-modsecurity-ruleset#mitigate-brute-force-attacks)
    - [Time Span](https://github.com/theMiddleBlue/wordpress-modsecurity-ruleset#time-span)
    - [Threshold](https://github.com/theMiddleBlue/wordpress-modsecurity-ruleset#threshold)
    - [Ban period](https://github.com/theMiddleBlue/wordpress-modsecurity-ruleset#ban-period)
  - [Block user enumeration](https://github.com/theMiddleBlue/wordpress-modsecurity-ruleset#user-enumeration)
  - [Block access to xmlrpc.php](https://github.com/theMiddleBlue/wordpress-modsecurity-ruleset#block-access-to-xmlrpcphp)
  - [Log login and logout events](https://github.com/theMiddleBlue/wordpress-modsecurity-ruleset#log-login-and-logout-events)
- [Test Brute-Froce Attacks](https://github.com/theMiddleBlue/wordpress-modsecurity-ruleset#test-brute-force-attack)
- [Contribute!](https://github.com/theMiddleBlue/wordpress-modsecurity-ruleset#contribute)
- [Contacts](https://github.com/theMiddleBlue/wordpress-modsecurity-ruleset#contacts)

# Install
just clone this repository with `git clone https://github.com/theMiddleBlue/wordpress-modsecurity-ruleset.git`

### Use with OWASP CRS
This rule set is intended to be used with the OWASP CRS3. You just need to clone this repository and then includes it in your modsecurity.conf:

```perl
...

Include wordpress-modsecurity-ruleset/*.conf
```

Optionally you can change the default behavior as described in the 01_SETUP.conf file:

```perl
...

SecAction "phase:1,id:22000000,nolog,pass,t:none,setvar:tx.wprs_client_ip=%{REMOTE_ADDR}"
SecAction "id:22000004,phase:1,nolog,pass,t:none,setvar:tx.wprs_check_bruteforce=1"
SecAction "id:22000005,phase:1,nolog,pass,t:none,setvar:tx.wprs_bruteforce_timespan=120"
SecAction "id:22000010,phase:1,nolog,pass,t:none,setvar:tx.wprs_bruteforce_threshold=5"
SecAction "id:22000015,phase:1,nolog,pass,t:none,setvar:tx.wprs_bruteforce_banperiod=300"
SecAction "id:22000020,phase:1,nolog,pass,t:none,setvar:tx.wprs_log_authentications=1"
SecAction "id:22000025,phase:1,nolog,pass,t:none,setvar:tx.wprs_allow_xmlrpc=0"
SecAction "id:22000030,phase:1,nolog,pass,t:none,setvar:tx.wprs_allow_user_enumeration=0"

Include wordpress-modsecurity-ruleset/*.conf
```

# Configurations

### Real Client IP Address
**Rule 22000000: Client IP Address.** This rule set the "real" client IP Address.
This usually is `%{REMOTE_ADDR}` but when you are behind CloudFlare or a Load Balancer,
the user's IP Address is inside a header parameter like X-Forwarded-For, or True-Client-IP,
or CF-Connecting-IP for CloudFlare. See the `01-SETUP.conf` file for more information:

```perl
SecAction "phase:1,id:22000000,nolog,pass,t:none,setvar:tx.wprs_client_ip=%{REMOTE_ADDR}"
```

### Mitigate Brute-Force Attacks
**Rule 22000004: Enable / Disable Brute-force mitigation.**
When `wprs_check_bruteforce` is set to `1` WPRS will try to mitigate brute-force attacks.
By default it will check if a user performs more then 5 login attempts, in a time span of 2 minutes,
and blocks it for 5 minutes.

```
setvar:tx.wprs_check_bruteforce=1  =  brute-force mitigation enabled
setvar:tx.wprs_check_bruteforce=0  =  brute-force mitigation disabled
```

default: 1

```perl
SecAction "id:22000004,phase:1,nolog,pass,t:none,setvar:tx.wprs_check_bruteforce=1"
```

### Time Span
**Rule 22000005: Time Span.**
How many seconds the login counter will be incremented
on each login attempt on /wp-login.php. For example, if you
want to increment the login attempt counter for a 10 minutes span:

```
setvar:tx.wprs_bruteforce_timespan=600
```

default: 120 (2 minutes)

```perl
SecAction "id:22000005,phase:1,nolog,pass,t:none,setvar:tx.wprs_bruteforce_timespan=120"
```

### Threshold
**Rule 22000010: Threshold.**
This rule set how many login attempts (inside the time span period) WPRS will accepts before ban.
For example, if you set this to 10, WPRS will ban the user at the 11th attempt.

```
setvar:tx.wprs_bruteforce_threshold=10
```

default: 5

```perl
SecAction "id:22000010,phase:1,nolog,pass,t:none,setvar:tx.wprs_bruteforce_threshold=5"
```

### Ban period
**Rule 22000015: Ban period.**
This rule set for how long a user will be banned if a brute-force attempt is detected.
For example, if you want to block a user for 5 mins you'll set this to 300:

```
setvar:tx.wprs_bruteforce_banperiod=300
```

default: 300

```perl
SecAction "id:22000015,phase:1,nolog,pass,t:none,setvar:tx.wprs_bruteforce_banperiod=300"
```

### User enumeration
**Rule 22000030: User Enumeration.**
This rule enable or disable requests like "/?author=1".
An attacker could enumerate all active users by incrementing
the author parameter.

```
setvar:tx.wprs_allow_user_enumeration=1 = allows request like /?author=1
setvar:tx.wprs_allow_user_enumeration=0 = blocks request like /?author=1
```

default: 1

```perl
SecAction "id:22000030,phase:1,nolog,pass,t:none,setvar:tx.wprs_allow_user_enumeration=1"
```

### Block access to xmlrpc.php
**Rule 22000025: XMLRPC.**
This rule enable or disable access on xmlrpc.php script.
Usually many users doesn't use the xmlrpc.php but they leave it
active, and this could lead to a brute-force amplification attacks.

```
setvar:tx.wprs_allow_xmlrpc=1 = allows reuests to xmlrpc.php
setvar:tx.wprs_allow_xmlrpc=0 = blocks reuests to xmlrpc.php
```

default: 1

```perl
SecAction "id:22000025,phase:1,nolog,pass,t:none,setvar:tx.wprs_allow_xmlrpc=1"
```

### Log login and logout events
**Rule 22000020: Log authentication events.**
This rule enable or disable the logging of authentication events.
If you enable this, each time a user login on /wp-login.php a log is produced.

```
setvar:tx.wprs_log_authentications=1 = enables logging
setvar:tx.wprs_log_authentications=0 = disables logging
```

default: 1

```perl
SecAction "id:22000020,phase:1,nolog,pass,t:none,setvar:tx.wprs_log_authentications=1"
```

## Test Brute-Force Attack
Following a brute-force attack test using wpscan, against the admin user.
The WPRS ban the attacker IP and blocks all requests after 5 failed login:

[![asciicast](https://asciinema.org/a/192223.png)](https://asciinema.org/a/192223)


## Contribute!
Please, feel free to contribute by a Pull Request


## Contacts
theMiddle twitter account: [@AndreaTheMiddle](https://twitter.com/AndreaTheMiddle)<br>
Rev3rse Security twitter account (ITA): [@rev3rsesecurity](https://twitter.com/rev3rsesecurity)<br>
Rev3rse Security YouTube (ITA): https://www.youtube.com/rev3rsesecurity
