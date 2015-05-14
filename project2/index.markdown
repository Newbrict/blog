---
layout: post
title:  "rpisec_nuke binary exploitation writeup"
date:   2015-05-14 00:15:00
categories: urxvt terminal ssh
---

This semester at RPI I'm taking the class "Modern Binary Exploitation". This
post will detail how I reversed and cracked the **rpisec_nuke** binary.

## Understanding rpisec_nuke

Running the binary shows us 3 options, **KEY 1**, **KEY 2**, and **KEY 3**
<img src="{{ "/images/rpisec_nuke/menu.png" | prepend: site.baseurl }}"/>

**KEY 1** is seemingly the most simple, it asks us for a launch key, and then
tries to authenticate with that.
<img src="{{ "/images/rpisec_nuke/key_one.png" | prepend: site.baseurl }}"/>

**KEY 2** looks more complicated, it askes us for an 'AES-128 CRYPTO KEY',
a data length, and the data to encrypt.
<img src="{{ "/images/rpisec_nuke/key_two.png" | prepend: site.baseurl }}"/>

**KEY 3** is presumably to be the most complicated. it starts by asking us to
confirm the launch session, which is found at the top of the banner, then it
gives us a challenge and asks for a response.
<img src="{{ "/images/rpisec_nuke/key_three.png" | prepend: site.baseurl }}"/>

## Unlocking **KEY 1**
After some time in IDA I found a fantastically simple bug that lets us get
through all the authentication steps. If I input null bytes then it stops reading
with length 0, with this the strncomp passes since all of the ( zero ) characters
were matches.

## Unlocking **KEY 2** and determining PRNG
After spending a bunch of time in IDA looking through two, I couldn't find the vuln.
I gave up and went to key three. The first thing I noticed is that if you fail the
launch session on the first time you open three it frees that struct on the WOPR.
In GDB I could see that going back into two after this occurance causes it to allocate
the struct right on top of where three's free'd one was. If I enter in garbage on two
and then go back to three the challenge that I see will contain the password for two
xor'd with random integers.

The first step here was to figure out the prng seed. It was easy to find using IDA
that it was seeded with the current time, and since in key three they give us the time
of access we can use this as a reference point. If I just enter new lines during key two
then when I get the key three, the first line of the challenge after uxoring should be

{% highlight bash %}
0a00000000000000000000000000000000
{% endhighlight %}

I wrote a loop to keep trying times lesser than the time output in key three until it
was able to unxor to this value. Once I had this I knew the prng seed and could take
advantage of any random numbers in the program. I also used this seed to unxor key two
which gave me the password in plain text :) Entering this key in unlocks the level.

{% highlight bash %}
key2 = 4e96e75bd2912e31f3234f6828a4a897
{% endhighlight %}

## Unlocking **KEY 3**
At this point I can guess random numbers. I spent some time looking at what the
challenge actually does: Each time you open key three it xors the challenge with
random numbers, then xors the diagonal 4 bytes at a time with 4 bytes of the password.
when it checks your challenge it does the same thing agian.  since I already can
undo the random number xor, figuring out the password was as simple as unxoring
the random numbers.

{% highlight bash %}
key3 =
90357303000000000000000000000000
00000000412ef8220000000000000000
0000000000000000cd10e79800000000
000000000000000000000000760491a6
{% endhighlight %}

To unlock the level I simple xor the challenge with the next 16 random numbers
and also with the block above.

<img src="{{ "/images/rpisec_nuke/unlocked.png" | prepend: site.baseurl }}"/>

## Programming Nuke86
The first thing I see when pressing the almighty 4 is a prompt asking me for the
targetting code. After I give my input nuke86 tells me the checksum and then fails.
The first thing to do is to figure out how to pass this step. In IDA I see that the
checksum is compared against the xor of all three key flags which represent whether
or not you have unlocked that key.

{% highlight bash %}
checksum = 0xCAC380CD ^ 0xBADC0DED ^ 0xACC3D489
checksum = 0xDCDC59A9
{% endhighlight %}

Also in the compute_checksum function I see that the checksum is just the composite
of my input xor'd 4 bytes at a time. I played around with my input until I was able
to get it to match **0xDCDC59A9** This showed me "PROGRAMMING COMPLETE". When I
then type launch it shows me mission failed shutting down, etc. Now it's time to
really learn how to program this thing.

In IDA launch_nuke reveals the "programming language" used by nuke86. Here's a mapping
of the instructions:

{% highlight text %}
R = Reprogram Nuke
D = Compare to DOOM and DISARM, if DOOM then detonate, if DISARM, then disarm.
E = Compare to END, end the program.
I = Increment the "program pointer"
O = Output the "Targetting status code" which is relative to where the "program pointer" is
S = Set the character at the "program pointer" to the next character
{% endhighlight %}

With this it's very easy to write a program to detonate on general doom, just
use S and I to write it in, then type DOOM to blow it up

<img src="{{ "/images/rpisec_nuke/doom_boom.png" | prepend: site.baseurl }}"/>

The exploit was pretty trivial to find, if I just keep typing I, then the "program pointer"
goes way overboard and gives me write access to the function pointers in the struct
namely the detonate_nuke pointer. So I can just I until I'm there, then put in my rop
chain and DOOM to detonate. I tested this using strace to see if I can set the real
program pointer to **0x41414141** and I was successful.

<img src="{{ "/images/rpisec_nuke/seg.png" | prepend: site.baseurl }}"/>

Since I already have a pointer to the WOPR I can just use that to point to where
/bin/sh is input within the nuke86 program, and all I have to do is put in my rop
chain after I do all my offsetting with **I**. I did not actually do the rop chain
but I have a high suspicion that it will be the same execve setup that we've done
many times.

My Auth Program can be found here:
[auth.py](https://gist.github.com/Newbrict/e78158a27600bc6329de)

My Key2 && Key3 inital finders can be found here:
[keys.py](https://gist.github.com/Newbrict/b0c8cbefa360f6d52379)
