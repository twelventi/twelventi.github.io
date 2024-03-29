---
layout: post
title:  "WiFi Coffee Makers are only a Little More Secure than I Thought" # This is the title that is displayed to users
date:   2021-03-31 00:00:00 -0500 # Remember to set this the same as the filename to avoid confusion
categories: jekyll update # Don't mess with categories unless you know how Jekyll works
author: David Bono
# Uncomment to attach downloads
# downloads:
#     - file_url: /assets/downloads/example_file.pdf
#       name: Example File
---

*This is a repost article originally posted on [fordhamcss.org](https://fordhamcss.org/blog/2021/03/31/wifi-coffee-makers-are-only-a-little-more-secure-than-I-thought.html). I wrote and posted it in both places but I figured I'd also keep it here for historical purposes*

Coming off our recent [Hacking a Coffee Maker Event](https://fordhamcss.org/events/2021/03/22/hacking-a-coffee-maker.html/), I wanted to write up a blog post with more technical information about what we did. So here's kind of a summary of what I discovered about the coffee makers, and if you came to the event, what was actually going on when we were sending messages to our CaaS (Coffee as a Service) server.

# TLDR;

So very simply, the coffee makers are vulnerable to a replay attack if you can sniff the communication between a phone and the coffee maker when you send a  "brew" message to the coffee maker. You can resend the messages the phone sends in the same order, after a TCP connection is established, and the coffee maker will turn on. I started trying to reverse engineer the binary protocol before I realized this.

# Background

## Motivation

The motivation for this research stemmed from IoT devices not having the greatest reputation when it comes to security. If you're unaware about some of the poor decisions that have been made when designing IoT devices, then check out [this](https://www.youtube.com/watch?v=h5PRvBpLuJs). So anyway, I found these Atomi "Smart" Coffee Makers (When talking about "smart" devices, I feel that "smart" should always be put in quotes) that work over WiFi, and allow you to brew coffee from your phone. So that's great, TCP/IP, networking, and packet sniffing!

## Setup

I had to register the smart coffee maker to an Atomi account in order to access the functionality where you can turn it on from a phone. After all, this was my goal. The registration process involved connecting to the WiFi network the coffee maker emits when it's turned on for the first time from your phone, and then setting some configuration in the Atomi Smart app. I tried to exploit the setup process for this, however I didn't have too much luck sniffing packets or port scanning the network the coffee maker emits at this stage, although I didn't spend too much time on this. If I were to look at this again, I would probably launch an Android emulator on my computer to have the Atomi app downloaded, and use Wireshark at this step to sniff the configuration packets. 

I moved on to the actual "brew coffee from your phone" feature. I setup a Raspberry Pi wireless access point, so I could sniff all traffic that gets sent to and from the coffee maker. This can be done numerous ways but my solution was the Raspberry Pi with PiOS and TCP dump. I also was able to get the packet dump to show nicely on my laptop in Wireshark by using this command: 

```
ssh root@raspberrypi tcpdump -i wlan0 -U -s0 -w - 'not port 22' | "C:\Program Files\Wireshark\Wireshark.exe" -k -i -
```

After connecting both my phone and the coffee maker to my Raspberry Pi WAP (wireless access point), I was able to see the conversations the coffee maker and phone were sending back and forth when a "brew" message was sent. 

The end goal with this setup is to simply learn "How does this device work?". If you are a security researcher and you're looking for vulnerabilities in a certain system, if you can understand how the system works, you can isolate the weak points and target your research at those areas. 

# Discoveries

Now to start examining what is going on... The messages that do not have an indent before them were sent from my phone to the coffee maker. The ones that are tabbed over were the responses from the coffee maker. 

```
00000000  00 00 55 aa 00 00 00 01  00 00 00 0a 00 00 00 58   ..U..... .......X
00000010  df e8 53 b4 31 f5 9e 52  0c 95 a0 db be 60 64 a2   ..S.1..R .....`d.
00000020  19 59 a3 1d f6 78 8d 71  da d4 33 40 08 3a 16 38   .Y...x.q ..3@.:.8
00000030  90 4c 1a 03 a0 5c b1 bd  79 d4 e0 d6 67 22 91 95   .L...\.. y...g"..
00000040  19 59 a3 1d f6 78 8d 71  da d4 33 40 08 3a 16 38   .Y...x.q ..3@.:.8
00000050  39 29 02 a5 50 92 1e 67  df 43 ff df ee b0 89 01   9)..P..g .C......
00000060  d0 59 17 5f 00 00 aa 55                            .Y._...U 
    00000000  00 00 55 aa 00 00 00 01  00 00 00 0a 00 00 00 2c   ..U..... .......,
    00000010  00 00 00 01 aa 05 53 2e  e0 13 aa ab e2 7b 0c 31   ......S. .....{.1
    00000020  fe fc e9 17 e0 18 e9 31  f2 05 86 09 e7 b1 75 55   .......1 ......uU
    00000030  d2 29 34 47 1c c3 81 68  00 00 aa 55               .)4G...h ...U
00000068  00 00 55 aa 00 00 00 02  00 00 00 09 00 00 00 08   ..U..... ........
00000078  4b 65 24 a4 00 00 aa 55                            Ke$....U 
    0000003C  00 00 55 aa 00 00 00 00  00 00 00 09 00 00 00 0c   ..U..... ........
    0000004C  00 00 00 00 b0 51 ab 03  00 00 aa 55               .....Q.. ...U
00000080  00 00 55 aa 00 00 00 03  00 00 00 09 00 00 00 08   ..U..... ........
00000090  5c 1e 30 e7 00 00 aa 55                            \.0....U 
    00000058  00 00 55 aa 00 00 00 00  00 00 00 09 00 00 00 0c   ..U..... ........
    00000068  00 00 00 00 b0 51 ab 03  00 00 aa 55               .....Q.. ...U                       
00000098  00 00 55 aa 00 00 00 04  00 00 00 07 00 00 00 87   ..U..... ........      <>     This is the
000000A8  33 2e 33 00 00 00 00 00  00 00 01 01 72 f4 e0 fb   3.3..... ....r...    <>      message that
000000B8  c5 a4 be bf 3d 35 28 61  8e ca 37 fb c3 bd 51 aa   ....=5(a ..7...Q.  <> ######### tells the 
000000C8  62 51 89 70 c5 7f 52 89  67 4d b6 77 78 83 6f fa   bQ.p..R. gM.wx.o.    <>      coffee maker
000000D8  4f 32 40 dd a3 1f d4 17  a4 cb 38 b7 8f 7c 0b 6e   O2@..... ..8..|.n      <>    to turn on
000000E8  2b e4 c4 42 de 3e 0f c7  95 09 56 7a d8 03 ac e7   +..B.>.. ..Vz....
000000F8  d1 dd 48 a1 f0 b9 a4 c3  17 d2 7d b5 8d 29 6b 00   ..H..... ..}..)k.
00000108  34 31 f6 7b ea c6 a4 28  b1 a8 85 e8 00 7f 78 e8   41.{...( ......x.
00000118  dd 48 be 84 9b 05 1c a9  14 de eb 34 96 a9 44 e3   .H...... ...4..D.
00000128  36 36 6a 00 00 aa 55                               66j...U
    00000074  00 00 55 aa 00 00 00 04  00 00 00 07 00 00 00 0c   ..U..... ........
    00000084  00 00 00 00 b8 2a 1a 07  00 00 aa 55               .....*.. ...U
    00000090  00 00 55 aa 00 00 00 00  00 00 00 08 00 00 00 4b   ..U..... .......K
    000000A0  00 00 00 00 33 2e 33 00  00 00 00 00 00 00 29 00   ....3.3. ......).
    000000B0  00 00 01 fb c5 a4 be bf  3d 35 28 61 8e ca 37 fb   ........ =5(a..7.
    000000C0  c3 bd 51 f9 03 10 4b 91  31 ef 75 ef 00 4c f7 61   ..Q...K. 1.u..L.a
    000000D0  41 3f 1b e8 dd 48 be 84  9b 05 1c a9 14 de eb 34   A?...H.. .......4
    000000E0  96 a9 44 e5 93 7d 50 00  00 aa 55 00 00 55 aa 00   ..D..}P. ..U..U..
    000000F0  00 00 00 00 00 00 08 00  00 00 4b 00 00 00 00 33   ........ ..K....3
    00000100  2e 33 00 00 00 00 00 00  00 2a 00 00 00 01 f5 31   .3...... .*.....1
    00000110  e0 e8 c3 66 57 a6 f8 82  3e ce 5f 69 0c 4d 36 e5   ...fW... >._i.M6.
    00000120  b4 4d ce 97 b2 27 16 56  78 de 1e ee c5 ce 51 c7   .M...'.V x.....Q.
    00000130  ac 34 bf 9d 24 37 f9 66  7d 24 b7 25 11 f1 cb 9e   .4..$7.f }$.%....
    00000140  86 24 00 00 aa 55 00 00  55 aa 00 00 00 00 00 00   .$...U.. U.......
    00000150  00 08 00 00 00 4b 00 00  00 00 33 2e 33 00 00 00   .....K.. ..3.3...
    00000160  00 00 00 00 2b 00 00 00  01 09 1f fd 19 03 e7 53   ....+... .......S
    00000170  c1 ae b7 96 73 8b 84 26  54 4b 9b 13 2b 46 35 1a   ....s..& TK..+F5.
    00000180  5d 2a ec f8 03 08 2f bf  0f 97 b1 c1 2f df da 07   ]*..../. ..../...
    00000190  e1 07 e7 38 73 3f 21 22  c8 1c d2 21 19 00 00 aa   ...8s?!" ...!....
    000001A0  55                                                 U
0000012F  00 00 55 aa 00 00 00 05  00 00 00 07 00 00 00 87   ..U..... ........
0000013F  33 2e 33 00 00 00 00 00  00 00 02 01 72 f4 e0 8a   3.3..... ....r...
0000014F  fd 4f 07 70 f4 6a d3 e4  cb bb 04 57 4f d1 fc 3f   .O.p.j.. ...WO..?
0000015F  d1 39 3f ec e0 25 9f fa  d8 81 e6 15 20 f1 99 c8   .9?..%.. .... ...
0000016F  fe 43 1b 80 31 31 95 01  bc 15 a9 56 00 bc 93 e8   .C..11.. ...V....
0000017F  28 9f cc d5 ec 55 72 6a  18 aa 8d 2f 9f e8 14 8e   (....Urj .../....
0000018F  c2 7d fd de bb af 95 f6  8c 20 0a dc f4 30 36 e6   .}...... . ...06.
0000019F  7f 13 16 23 99 3a 27 41  42 3a 7b 63 60 68 69 39   ...#.:'A B:{c`hi9
000001AF  29 02 a5 50 92 1e 67 df  43 ff df ee b0 89 01 0c   )..P..g. C.......
000001BF  18 06 89 00 00 aa 55                               ......U
    000001A1  00 00 55 aa 00 00 00 05  00 00 00 07 00 00 00 0c   ..U..... ........
    000001B1  00 00 00 00 65 bc c3 82  00 00 aa 55               ....e... ...U
    000001BD  00 00 55 aa 00 00 00 00  00 00 00 08 00 00 00 4b   ..U..... .......K
    000001CD  00 00 00 00 33 2e 33 00  00 00 00 00 00 00 2c 00   ....3.3. ......,.
    000001DD  00 00 01 8a fd 4f 07 70  f4 6a d3 e4 cb bb 04 57   .....O.p .j.....W
    000001ED  4f d1 fc e3 98 77 3e e7  0e fd cc 5f 08 8f 81 d9   O....w>. ..._....
    000001FD  e8 61 1a 0f cf bf 62 6a  47 b1 a5 87 fa e5 5a 1b   .a....bj G.....Z.
    0000020D  5d 3e 8f 20 7f b5 57 00  00 aa 55                  ]>. ..W. ..U
    00000218  00 00 55 aa 00 00 00 00  00 00 00 08 00 00 00 4b   ..U..... .......K
    00000228  00 00 00 00 33 2e 33 00  00 00 00 00 00 00 2d 00   ....3.3. ......-.
    00000238  00 00 01 f5 31 e0 e8 c3  66 57 a6 f8 82 3e ce 5f   ....1... fW...>._
    00000248  69 0c 4d f0 33 e3 07 5e  63 e4 90 f8 96 39 36 ea   i.M.3..^ c....96.
    00000258  cb 27 91 b6 dd 73 10 7a  5e f1 05 87 e9 39 da c3   .'...s.z ^....9..
    00000268  bb e9 d8 38 8f 43 df 00  00 aa 55 00 00 55 aa 00   ...8.C.. ..U..U..
    00000278  00 00 00 00 00 00 08 00  00 00 4b 00 00 00 00 33   ........ ..K....3
    00000288  2e 33 00 00 00 00 00 00  00 2e 00 00 00 01 09 1f   .3...... ........
    00000298  fd 19 03 e7 53 c1 ae b7  96 73 8b 84 26 54 41 17   ....S... .s..&TA.
    000002A8  6f 67 d7 e4 11 ed 92 df  f0 40 47 a7 57 a0 97 42   og...... .@G.W..B
    000002B8  10 8c 08 ed 07 5d a5 16  60 69 e7 f6 22 67 25 8a   .....].. `i.."g%.
    000002C8  90 8d 00 00 aa 55                                  .....U
000001C6  00 00 55 aa 00 00 00 06  00 00 00 09 00 00 00 08   ..U..... ........
000001D6  16 89 75 a8 00 00 aa 55                            ..u....U 
    000002CE  00 00 55 aa 00 00 00 00  00 00 00 09 00 00 00 0c   ..U..... ........
    000002DE  00 00 00 00 b0 51 ab 03  00 00 aa 55               .....Q.. ...U
```

Now, if you've never seen any binary messages before you probably have a headache at this point, but let me break down some of these messages. 

The first message that the phone sends is a "hello" message. After doing some further testing, I discovered that this message contains some information that is unique to the registration of the coffee maker with your atomi account. 

The first response is also included, but tabbed over. If you notice, there's a similarity between both of these messages. The first four bytes of both message are `00 00 55 aa` and the last four are `00 00 aa 55`. If you look back at the other messages, you can see this pattern for every single message that gets sent both to and from the coffee maker. 

```
00000000  00 00 55 aa 00 00 00 01  00 00 00 0a 00 00 00 58   ..U..... .......X
00000010  df e8 53 b4 31 f5 9e 52  0c 95 a0 db be 60 64 a2   ..S.1..R .....`d.
00000020  19 59 a3 1d f6 78 8d 71  da d4 33 40 08 3a 16 38   .Y...x.q ..3@.:.8
00000030  90 4c 1a 03 a0 5c b1 bd  79 d4 e0 d6 67 22 91 95   .L...\.. y...g"..
00000040  19 59 a3 1d f6 78 8d 71  da d4 33 40 08 3a 16 38   .Y...x.q ..3@.:.8
00000050  39 29 02 a5 50 92 1e 67  df 43 ff df ee b0 89 01   9)..P..g .C......
00000060  d0 59 17 5f 00 00 aa 55
    00000000  00 00 55 aa 00 00 00 01  00 00 00 0a 00 00 00 2c   ..U..... .......,
    00000010  00 00 00 01 aa 05 53 2e  e0 13 aa ab e2 7b 0c 31   ......S. .....{.1
    00000020  fe fc e9 17 e0 18 e9 31  f2 05 86 09 e7 b1 75 55   .......1 ......uU
    00000030  d2 29 34 47 1c c3 81 68  00 00 aa 55               .)4G...h ...U
```

The next few messages appear to be some sort of keep alive messages and responses, however, they can provide some insight into the nature of this communication protocol, since they are so short. 

```
00000068  00 00 55 aa 00 00 00 02  00 00 00 09 00 00 00 08   ..U..... ........
00000078  4b 65 24 a4 00 00 aa 55                            Ke$....U 
    0000003C  00 00 55 aa 00 00 00 00  00 00 00 09 00 00 00 0c   ..U..... ........
    0000004C  00 00 00 00 b0 51 ab 03  00 00 aa 55               .....Q.. ...U
00000080  00 00 55 aa 00 00 00 03  00 00 00 09 00 00 00 08   ..U..... ........
00000090  5c 1e 30 e7 00 00 aa 55                            \.0....U 
    00000058  00 00 55 aa 00 00 00 00  00 00 00 09 00 00 00 0c   ..U..... ........
    00000068  00 00 00 00 b0 51 ab 03  00 00 aa 55               .....Q.. ...U    
```

The next four bytes after the start of the message are a message count. If you look at the entire transcript of messages, you'll notice that this increases by one every time a new message is sent. The next 8 bytes seem to be some kind of message type identifier. `00 00 00 09 00 00 00 08` shows up in both keep alive messages, and is also different for other messages. Those bytes; `00 00 00 09 00 00 00 08` must mean that the message is a keep alive message. The response from the coffee maker gives us a message type of: `00 00 00 09 00 00 00 0c`, so this must mean an "acknowledgement" message of the keep alive from the phone. 

There are 8 bytes left in the keep alive message. We recognize the `00 00 aa 55` as the ending bytes from the opening messages, which leaves us with `4b 65 24 a4` to figure out. I think this is some sort of checksum message, as these four bytes change in the next keep alive message. I never successfully reverse engineered this checksum value, however if I were to keep researching this, I would first start by capturing about 5 minutes of keep alive messages. 

After these messages comes the most interesting message; the "Brew" message. 

```
00000098  00 00 55 aa 00 00 00 04  00 00 00 07 00 00 00 87   ..U..... ........      <>     This is the
000000A8  33 2e 33 00 00 00 00 00  00 00 01 01 72 f4 e0 fb   3.3..... ....r...    <>      message that
000000B8  c5 a4 be bf 3d 35 28 61  8e ca 37 fb c3 bd 51 aa   ....=5(a ..7...Q.  <> ######### tells the 
000000C8  62 51 89 70 c5 7f 52 89  67 4d b6 77 78 83 6f fa   bQ.p..R. gM.wx.o.    <>      coffee maker
000000D8  4f 32 40 dd a3 1f d4 17  a4 cb 38 b7 8f 7c 0b 6e   O2@..... ..8..|.n      <>    to turn on
000000E8  2b e4 c4 42 de 3e 0f c7  95 09 56 7a d8 03 ac e7   +..B.>.. ..Vz....
000000F8  d1 dd 48 a1 f0 b9 a4 c3  17 d2 7d b5 8d 29 6b 00   ..H..... ..}..)k.
00000108  34 31 f6 7b ea c6 a4 28  b1 a8 85 e8 00 7f 78 e8   41.{...( ......x.
00000118  dd 48 be 84 9b 05 1c a9  14 de eb 34 96 a9 44 e3   .H...... ...4..D.
00000128  36 36 6a 00 00 aa 55
```

You can observe the patterns from the previous messages here. The start and end bytes; `00 00 55 aa`, and `00 00 aa 55`. There's the message count: `00 00 00 04`, and then the "brew" message type: `00 00 00 07 00 00 00 87`. The parts in the middle are some kind of data. I'm unsure what it means but this might be able to be reversed by observing what this message looks like when it's at different parts in the sequence (I.e. what happens when you send the "brew" message from your phone when it's the 5th packet in the sequence, 6th, etc)

This is where the messages started to get confusing. At this point, now that I had a basic understanding of what each message did, I started testing sending this data to the coffee maker. I did this with a simple python script that is linked in the original event post. 

Sending solely the "brew" message to the coffee maker didn't turn it on, however, I tried sending all of the client messages in order from python and boom! The coffee maker started brewing. Pwned: ✔

Since we had multiple coffee makers, I wanted to see what the scope of this replay attack was, and it turns out that something in the first message establishes the TCP connection as a valid TCP connection. Sending that first welcome message to different coffee maker on the network does not produce the "start brewing" result that we are seeking. This is where the coffee maker is actually more secure than I thought it might be. In order to turn on the coffee maker by sending network packets from python, you need to know the values in that original first message that establishes the communication as valid. The only way (I know of) doing this is by sniffing the communication between your phone and the coffee maker. If someone was in a position to do this on your home network, you may have bigger issues than your coffee maker magically turning on at 3 am. 

Another observation I made when testing was in regards to the "WiFi" button on the physical coffee maker. There's a button on the front panel that when unlit, disables all of the remote coffee functions. Naturally, you might think that this would disable the wireless card and disconnect the coffee maker from the network, however this is not the case. The WiFi button simply disables the firmware on the microcontroller from interacting with the coffee maker. If you've disabled the WiFi with the button, the coffee maker still accepts network requests and sends back the same data, however it doesn't turn the coffee maker on. 

At this point, I stopped trying to reverse the coffee maker, because I had enough substance to run our CSS event.

# Conclusion

In conclusion, there is a replay attack vulnerability that allows you to successfully turn on one of these coffee makers from python while the coffee maker is on a network. However, they have some security features that limit its scope to the messages that were sent to and from one specific coffee maker to a phone. There's definitely more that can be researched and if I were to continue researching this, I would sniff many more diverse phone-to-coffee maker conversations (such as a lengthy keep-alive sequence, and selecting "brew" at different points in the conversation to see how the checksums differ, etc) and further attempt to reverse engineer the binary protocol so that I could craft my own coffee maker packets. 

This was pretty much how the event worked (aside from exposing the coffee makers to the public internet so people who were remote at the event possibly could have recreated this behavior....). Thanks to all who came to the event, and I hope you were able to learn something about embedded devices!



newlines go <br>
<br><br><br><br>
<br><br><br><br>
<br><br><br><br>
<br><br><br><br>