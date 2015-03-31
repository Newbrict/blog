---
layout: post
title:  "tw33tchainz binary exploitation writeup"
date:   2015-03-18 17:00:00
categories: urxvt terminal ssh
---

This semester at RPI I'm taking the class "Modern Binary Exploitation". This
post will detail how I reversed and cracked the **tw33tchainz** binary.

## Understanding tw33tchainz

When we run tw33tchainz the first thing we are presented with is some art and a 
prompt for our username, and salt. upon entering these values we are given a
generated password.
<img src="{{ "/images/tw33tchainz/intro.png" | prepend: site.baseurl }}"/>

afterwards we are given a menu with 4 options, labelled 1, 2, 4, and 5. the
option labelled '3' is missing. Naturally I entered into the prompt 3, then I
was asked to enter a password, my guess was incorrect.

<img src="{{ "/images/tw33tchainz/menu.png" | prepend: site.baseurl }}"/>

Option 1 lets us enter a tweet which gets stored somewhere, and is appended to
the penguin's beak the next time the menu is output to our term.

<img src="{{ "/images/tw33tchainz/tweet.png" | prepend: site.baseurl }}"/>

Option 2 lets us view all of our previous tweets.

Option 4 simply prints out the ascii art that's on top of the menu. ( looks like
a mallet smashing a sombrero, or maybe a bird? nah... )

Option 5 quits the program.

## Digging deeper

I decided a good first step would be to map out all the functions defined in the
binary, I did this through the use of gdb

{% highlight bash %}
project1@warzone:/levels/project1$ echo "disas main" | gdbpeda tw33tchainz | grep "call" | grep -v "@plt"
   0x08048e7f <+155>:	call   0x8048fac <print_banner>
   0x08048e84 <+160>:	call   0x804886c <gen_pass>
   0x08048e89 <+165>:	call   0x8048923 <gen_user>
   0x08048eb4 <+208>:	call   0x8048fac <print_banner>
   0x08048eb9 <+213>:	call   0x8048a34 <print_menu>
   0x08048f27 <+323>:	call   0x8048c0e <do_tweet>
   0x08048f2e <+330>:	call   0x8048bcd <view_chainz>
   0x08048f35 <+337>:	call   0x8048d40 <maybe_admin>
   0x08048f48 <+356>:	call   0x8048fac <print_banner>
   0x08048f4f <+363>:	call   0x8048fc1 <print_exit>
{% endhighlight %}
<br>


I then ran the same command for each function in the output.

a quick checksec shows us that RELRO isn't full, therefore we can overwrite to
the GOT.
{% highlight text %}
CANARY    : disabled
FORTIFY   : disabled
NX        : disabled
PIE       : disabled
RELRO     : Partial
{% endhighlight %}
<br>

From here I wanted to see if there were any vulnerable printf statements that
would let me make arbitrary memory writes.

I was looking for printfs that touched user input, in the entire program there
are only two locations where these reside: **print_menu**, and **view_chainz**.
I started by going through each printf statement in **print_menu** and surely
enough there is printf which takes the previous tweet and uses it as the first
argument, a cardinal sin.

Unfortunately the block of code containing the vulnerable printf looks like
this:

{% highlight asm %}
0x08048a84 <+80>:	movzx  eax,BYTE PTR ds:0x804c0a8
0x08048a8b <+87>:	test   al,al
0x08048a8d <+89>:	je     0x8048ab4 <print_menu+128>
0x08048a8f <+91>:	mov    DWORD PTR [esp],0x8049143
0x08048a96 <+98>:	call   0x80485d0 <printf@plt>
0x08048a9b <+103>:	lea    eax,[ebp-0x19]
0x08048a9e <+106>:	mov    DWORD PTR [esp],eax
0x08048aa1 <+109>:	call   0x80485d0 <printf@plt>
{% endhighlight %}
<br>

the first three lines there check to see if you are admin. If **0x804c0a8** is
0 then you are not admin, if either of the bottom 2 bytes are non-zero then you
are considered an admin.

the vulnerable printf cannot be reached unless we're set to admin. So the next
step is to figure out what the correct password for option 3 is.

taking a look at the **gen_pass** function I see that it simply reads 16 bytes
from /dev/urandom, these 16 random bytes are the password required for option
3.

**gen_user** does the following

1. memset the salt with 16 bytes of 0xba
2. memset the user with 16 bytes of 0xcc
3. read 16 bytes into user
4. read 16 bytes into salt
5. call the hash function

**hash** then creates the generated password like so

1. byte wise addition between the salt and the secret password
2. xors that new value with the user

then later down in **gen_user** it calls **print_pass** which outputs the
generated password as hex.

in **print_pass**
{% highlight asm %}
0x08048851 <+52>:	mov    DWORD PTR [esp],0x8049070
0x08048858 <+59>:	call   0x80485d0 <printf@plt>
{% endhighlight %}
<br>
value at **0x08049070**
{% highlight asm %}
0x08049070 : 25 30 38 78 25 30 38 78 25 30 38 78 25 30 38 78   %08x%08x%08x%08x
{% endhighlight %}
<br>

given this, with a carefully chosen user and salt, we can easily recover the
secret pass from the generated pass that we are given.

I chose to use 15 of 0x01 for the user and 15 of 0xff for the salt, I'm using 15
instead of 16 because in gdb I saw that fgets adds a null terminator as the 16th
byte in each.

with these values I know that when we're adding the secret password to 0xff the
value will overflow and be 1 less than the original value.

afterwards when we xor this with the value 0x01 that will turn even numbers odd,
one greater than the original value, and odd numbers even, one less than the
original value.

for example if the secret byte was 0x55, when that's added to 0xff we end up with
the value 0x54, which then turns back into 0x55 after we xor it with 0x01.

further if the secret byte was 0x56, when that's added to 0xff we end up with
the value 0x55, which turns into 0x54 after we xor it with 0x01.

this means that in our generated password, we simply add 2 to the even bytes, and
leave the odd bytes unchanged. except for the last byte, which was null. The last
byte remains unchanged from the secret password.

I wrote a python script which takes the generated password as a command line
argument and returns the secret password.

[decode_pass.py](https://gist.github.com/Newbrict/feab6f9f04de36dac453)

<br>
**Success!**, now that we're admin we have a menu option labelled 6, and more
importantly, access to the vulnerable printf statement in **print_menu**
<img src="{{ "/images/tw33tchainz/admin.png" | prepend: site.baseurl }}"/>

since tweets are limited to 16 characters, the format string for the exploit
must be this length.

in gdb I noticed that the stack is actually misalighned by 1 byte so the first
character of the format string will be used to realign it.

{% highlight bash%}
tweet="A"
{% endhighlight %}
This means we have 15 bytes remaining for an arbitrary write.

First thing's first, we need the address to write to, this will consume 4 more
bytes which leaves us 11 bytes for **%x** and **%n** to actually write the value.

With 11 bytes there is simply not enough room for enough **%x** tokens to get to
the address on the stack, which means I'll have to use direct parameter access.
with all the required characters our tweet format looks like this

{% highlight bash%}
tweet="A....%???x%#$hhn"
{% endhighlight %}
**....** will be the address

The only thing left is to determine what values to substitute for **???**, and
which parameter **#** will be.

looking at the stack I can see that **#** has to be 8.

when doing some test writes in gdb I can see that whatever value I put in for
**???** will be written to memory as 5 higher than the original value which
means I just simply subtract 5 from the value I want to write and then the
correct value will be written. This works unless the value I want to write is
less than 5 since I can't write a negative number of bytes. To mediate this I
simply add 256 to every value which lets it overflow cleanly into the correct
value.

The final anatomy of a malicious tweet which writes the value **0x10** to the
location **0x43434343** looks like this.

{% highlight bash%}
tweet="ACCCC%267x%8$hhn"
{% endhighlight %}

Now it's just a matter of finding a location to write my shellcode, then writing
it, overwriting exit to point to that location ( Thanks RELRO! ) and cat'ing the
password :)

I used the **execve("/bin/sh")** shellcode found
[here](http://shell-storm.org/shellcode/files/shellcode-811.php)

and formatted it to be more easily readable by my final script:
[binsh.hex](https://gist.github.com/Newbrict/99fe72f72326d392636d)

I wrote a python script to create the format string given the address to write
to, the tweet number, and the value to write:
[get_tweet.py](https://gist.github.com/Newbrict/d593e6f80ec39b78c075)

and my final driver to input all this to the program, and cat the password is
written in bash: [driver.sh](https://gist.github.com/Newbrict/cc327dc28a455ae62f29)

## Results

{% highlight bash %}
$ ./driver.sh
...
...
Enter Choice: m0_tw33ts_m0_ch4inz_n0_m0n3y
{% endhighlight %}
