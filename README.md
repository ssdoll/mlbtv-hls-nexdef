# Introduction #

This app uses cURL and OpenSSL to connect to MLB.tv and play/save baseball games in Linux (and probably other OS's).  I wrote this because I really love my baseball and I really love my baseball in HD.

The code isn't pretty but it gets the job done.  I tried implementing bitrate switching but I haven't have time to test it so it probably doesn't work.

You also **need** a valid and paid for MLB.tv account to make this work.


# How to use #

First you need to checkout the source and compile.  Remember to have cURL, OpenSSL, and libconfig installed on your system.

If you're under Ubuntu, install the following:

libcurl4-openssl-dev
libconfig8-dev

If you're under Gentoo:

emerge net-misc/curl dev-libs/libconfig dev-libs/openssl

Next, running 'make' where you check'ed the source to will compile it and generate an executable called **mlb**.

After that, you need to use this in conjunction with [mlbviewer](http://mlbviewer.sourceforge.net/).  After you've setup mlbviewer, you'll need to run the mlbplay.py app and get a Base64 encoded url:

python mlbplay.py v=nym n=1 nu=1

You may need to edit the mlbplay.py app at or around line 237 and comment out the sys.exit() :
```
        print 'An error occurred locating the media URL:'
        print g.error_str
        sys.exit()
```

become
```
        print 'An error occurred locating the media URL:'
        print g.error_str
        #sys.exit()
```

You may see an error about locating the media URL but this isn't a problem.  What you are concerned with is the Base64 encoded string.  This is what I get:

```
An error occurred locating the media URL:
Could not parse NexDef stream list.  Try alternate coverage.

RTSP/1.0 400 Bad Request
Server: DSS/6.0.3 (Build/526.3; Platform/Linux; Release/Darwin Streaming Server; State/Development; )
Cseq: 
Connection: Close


start_of_b64_str_axxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.....................xxxxxxxxxxxxxxxxxxxxxx
```

If you do not see the base64 string then you may need to comment out the sys.exit() above.


Now that you have the string, run the 'mlbhls' app passing in the base64 string with the -B param.  This example will save the mlb video stream to a file called output.ts:

```
./mlbhls -o output.ts -B start_of_b64_str_axxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.....................xxxxxxxxxxxxxxxxxxxxxx 
```

If all goes well, you will see:

```
[MLB] Reading cfg file: mlb.cfg
[MLB] Output file: tmp.ts
[MLB] Max. Bandwidth: 0(bps)
[MLB] Bandwidth Locking: 1
[MLB] Fetching Master URL...
[MLB] Current Priority: 7 (bw: 4500000)
[MLB] Playlist refresh time: 20 (s)
-- SZ: 0
[MLB] Setting stream start time to (epoch): 1309396084
[MLB] Start fetching video from line: 3677 (3702)
[MLB] bytes decrypted: 3517292 (6s) -- xx/yy/0001.ts (time: 1 -- 1)
[MLB] bytes decrypted: 7035148 (12s) -- xx/yy/0002.ts (time: 1 -- 2)

```

Then play output.ts with mplayer:

```
mplayer -demuxer lavf output.ts
```

of if you want to use vdpau:

```
mplayer -demuxer lavf -vo vdpau -vc ffh264vdpau output.ts
```


# Options #
-B = Base64 Encoded String

-o = Output filename.  You can use '-' for stdout.

-f = Which Block to start at.  If the game is still in progress and -f is 0, it will be the most current block (live).  If you specify non-zero, it will start reading from the beginning with each block being about 6 seconds, so -f 5 will start 30 seconds from when MLB.tv started streaming.

-b = Max bitrate to use, you should be able to choose from:
4500000, 3000000, 1800000, 1200000, 800000, 500000

-l = Seconds to wait for before spawning player cmd (6 = 1 block)

-c = Config file (see mlb.cfg)