---
title: 'HackPack CTF 2020: Time Window'
date: 2020-05-18
categories: [writeups]
author: None
---

> The evil server admin has a secret but will only give it to you if you can
> guess his coveted hidden value, but it keeps changing with time:
> https://timed-key.cha.hackpack.club

<!--more-->

Inspecting the HTML we see that after entering a code the function `c` is
called:

```html
<form onsubmit="javascript:c();return false;" action="javascript:c();">
```

`c` can be found in `__.js` which looks like this:

```javascript
var x, y, dn = Date.now,
    rc = function() {
        return Math.floor(107 + 19 * Math.random())
    },
    rn = new Math.seedrandom("0x42").int32(),
    rt = function() {
        let r = Math.floor(1 + 7 * Math.random()),
            e = [];
        for (; r > 0;) e.push(rc()), r--;
        return String.fromCharCode(...e)
    },
    rr = function(r, e) {
        for (; r > 0;) e.unshift(e.pop()), --r;
        return e.join("")
    },
    lr = function(r, e) {
        for (; r > 0;) e.push(e.shift()), --r;
        return e.join("")
    },
    cf = function(r) {
        try {
            let e = Array.from(atob(rr(243, Array.from(r)))),
                n = 1;
            for (let r = 0; r < e.length; r++) r === n && (e.splice(r + 1, +e[r + 1] + 1), n += n);
            return +lr(168, e)
        } catch (r) {}
        return -1
    },
    c = function() {
        let r = document.getElementById("message").value,
            e = document.getElementById("msg");
        x = rn % 2 == 0 ? lr : rr, y = rn % 2 != 0 ? lr : rr, cf(r) >= dn() - 6e4 ? fetch("/check", {
            method: "POST",
            mode: "same-origin",
            cache: "no-cache",
            headers: {
                "Content-Type": "application/json"
            },
            referrer: "no-refferer",
            body: JSON.stringify({
                key: r
            })
        }).then(r => r.json()).then(r => {
            r.hasOwnProperty("flag") ? e.innerHTML = "Congrats! <br/>" + r.flag : e.innerText = "Nope!!!"
        }).catch(r => {
            console.error(r)
        }) : e.innerText = "Nope!!!"
    };
```

`c` basically performs the check `cf(r) >= dn() - 6e4`, and if successful sends
the input to the server, which we can assume performs the same check before
returning the flag. `dn` is the current date, so we see that the input is not
especially time critical, we just need to make `cf(r)` large enough.

Reading the other functions, we see that `rc` returns a random character, `rn`
a random number, and `rt` a random short string. `rr` and `lr` are right and
left rotations. `cf` is more interesting, with the loop

```javascript
for (let r = 0; r < e.length; r++)
    r === n && (e.splice(r + 1, +e[r + 1] + 1), n += n)
```

at its heart. This loop looks at the characters in `e` at positions which are
one above powers of two, omitting the zeroth power. For each such position `i`,
it removes from `e` the corresponding character `e[i]`, along with the
`Number(e[i])` immediately following characters. What is left can be anything
that parses to a number.

We want this number to be large, but we might as well make it short – I choose
`9e99`. Now we counter the removal loop by inserting zeros (which will
effectively remove themselves) in appropriate positions, giving us `9e0909`.
We can now leverage the JavaScript console to back-form a code which maps to
our desired number

```javascript
» m = lr(243, Array.from(btoa('9e0909')))
← "wOTA5OWU"
```

and then enter it

```javascript
» document.getElementById('message').value = m; c()
```

to get the flag:

```
flag{'t1m3_w1nd0w_g3nk3y'}
```
