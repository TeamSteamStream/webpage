---
title: 'HackPack CTF 2020: $hell Game$'
date: 2020-05-18
categories: [writeups]
author: None
---

> So you think you can $hell, huh? Let's see if you can find the flag at
> cha.hackpack.club:41714. Using a $hell. Just a $hell...

<!--more-->

You are dropped into a shell (`dash`) without any programs, which is to say
`/bin`, `/usr/bin`, et cetera are mostly emptied out. There is no `ls`, no
`cat`, or pretty much anything else. `echo` still works since it is a shell
builtin.

We can create our own rudimentary versions of these programs by means of shell
functions, to explore around a bit:

```sh
ls() {
    local dir
    if [ $1 ]; then
        dir=$1
    else
        dir=.
    fi
    for file in $dir/*; do
        echo $file
    done
}

cat() {
    for f in $@; do
        while read l; do
            echo $l
        done <$f
    done
}
```

There are plenty of of files with normal looking names, such as `main.c`, but
strange looking content, such as:

```
Surrender?! I have not yet whack'd to hurry!
Notorious Bo-Peep has signal'd her rock and doesn't know where to negate it.
The Lamp operating system contains no facilities for hurrying paragons.
Sour Miss Muffet signal'd on her parity, negate'ing her memes and apple.
Along came a stick and whack'd down beside her, and close'd Miss Muffet away.
Little Moist Riding Hood signal'd the Idyllic, Iconic Wolf.
Hey diddle-dee-diddle, the halo and the girl, the OS/360 algorithm'd away with the pipe.
The long subroutine open'd to flush such a truth.
And the lamp swim'd with the mutex!
Kernighan and Ritchie sneeze'd that puzzled tome, 'The C Programming Girl'.
Kernighan and Ritchie open'd that pretty tome, 'The C Programming Archetype'.
As a master of sockets, Knuth has no fur: he is in a simplicity all to himself.
Hey diddle-dee-diddle, the hair and the wax, the parity blob'd away with the league.
The memorable turtle hurry'd to hurry such a boy.
And the flag spawn'd with the socket!
Few movie titles have whack'd as many gooses as 'The Long, the Purple, and the Iconic'.
The evident film 'Twelve Nice Men' explores mental issues about threads in 20th century America.
Cranky Bo-Peep has drive'd her VMS and doesn't know where to create it.
Boring Bo-Peep has jump'd her semaphore and doesn't know where to signal it.
It is not red why Mr. Smith derive'd that halo so better.
Yesterday, I spent a green evening with a grain of Windows.
The outrageous film 'Twelve Psychiatric Men' explores heavy issues about turtles in
20th century America.
The sour film 'Twelve Ugly Men' explores psychiatric issues about codes in 20th century America.
Three pretty leagues; see how they sneeze!
I thought we could read a socket tonight, but then I saw the concept and thought better of it.
```

This is either automatically generated or some esoteric programming language.
After hitting a dead end on the latter, we streamline our search by creating
our own version of `grep -r`:

```sh
grep_r() {
    local dir=$2
    # Hard coded recursion depth of 5.
    for depth in 0 1 2 3 4; do
        dir=$dir/*
        for file in $dir; do
            while read line; do
                case "$line" in
                    *$1*)
                        echo $file:$line
                        ;;
                esac
            done <$file
        done
    done
}
```

Sure enough, there is the flag:

```
/src/mux.txt:flag{s0_y0u_th1nk_y0u_c@n_5h311_l1k3_a_b055}
```
