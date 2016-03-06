# Technical Stuff #

If you care about how it works, keep reading!

# Overview #

I'm not sure when it happened but MLB.tv started using Apple's HTTP Live Streaming Service (aka HLS) to transmit games in HD.  Their web plugin (autobahn) is nothing but a RTMP to HLS proxy.

# HLS #

For those of you not familiar with HLS it's pretty simple.  In it's most basic form, it's just a audio/video stream muxed with a MPEG-TS container.    The reason (I think) for using MPEG-TS is that it's a pretty resilient transport container when you're dealing with a unstable and unknown factor (the internet) and the packet size is 188 bytes, which makes for easy splitting.  It also has continuity counters, SI tables, and a whole plethora of features that help recover a stream should it run into problem s and lose packets.

So, the audio/video is encoded and muxed into a MPEG-TS.  From here, the MPEG-TS is further split in small chunks, usually 6-10 seconds long and pushed to a regular web server.  During this process a playlist (m3u8) is also generated and contains links to chunks.

That's pretty much it.  If a someone wants to support multiple bitrates,
the need to create multiple MPEG-TS streams, with multiple playlists that are at the bitrates they want to support.  All these streams are put in a master playlist where a client can parse.

# Video #

The variety of MPEG-TS used in HLS only really supports MPEG1/2, MPEG4 (part2 and part10), and maybe VC-1.  H.264 (MPEG4 Part10) is the most popular since it's well supported and has good compression.  (This is what's used for MLB.tv)

# Audio #

MPEG-TS supports MP2/MP3, AC3, and AAC.  AAC is probably what's being used in most HLS setups. (And is being used by MLB.tv)

# Encryption #

TODO