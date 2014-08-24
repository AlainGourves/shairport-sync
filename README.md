Shairport Sync
=============
Shairport Sync allows you to synchronise the audio coming from all your devices. Specifically, Shairport Sync allows you to set the "latency" -- the time between when a sound is sent and when it is played. This allows you to synchronise Shairport Sync devices reliably with other devices playing the same source. For example, synchronised multi-room audio is possible without difficulty.

Shairport Sync is a pretty substantial rewrite of Shairport 1.0 by James Laird and others -- please see https://github.com/abrasive/shairport/blob/master/README.md#contributors-to-version-1x for a list of the contributors to Shairport 1.x and Shairport 0.x. From a "heritage" point of view, Shairport Sync is a fork of Shairport 1.0 and the active branch is called Shairport 2.1.

Shairport Sync  works only with Linux and ALSA. The sound card you use must be capable of working with 44,100 samples per second interleaved PCM stereo (you'll get a message in the logfile if there's a problem).

For more about the motivation behind Shairport Sync, please see the wiki at https://github.com/mikebrady/shairport-sync/wiki.

What is Shairport?
----------
Shairport emulates an AirPort Express for the purpose of streaming audio from iTunes and compatible iPods and iPhones. It implements a server for the Apple RAOP protocol.
Shairport does not support AirPlay video and photo streaming.

Shairport Sync does Audio Synchronisation
---------------------------
Shairport Sync allows you to set a delay (a "latency") from when music is sent by iTunes or your iOS device to when it is played in the Shairport Sync audio device. The latency can be set to match the latency of other output devices playing the music (or video in the case of the AppleTV or Quicktime), achieving audio synchronisation. Shairport Sync uses extra timing information to stay synchronised with the source's time signals, eliminating "drift", where audio streams slowly drift out of synchronisation.

To stay synchronised, if an output device is running slow, Shairport Sync will delete frames of audio to allow it to keep up; if the device is running fast, Shairport Sync will insert interpolated frames to keep time. The number of frames inserted or deleted is so small as to be almost  inaudible. Frames are inserted or deleted as necessary at pseudorandom intervals. Alternatively, Shairport Sync can resample the audio feed to ensure the output device can keep up. Resampling is even less obtrusive than insertion and deletion but requires a good deal of processing power � most embedded devices probably can't support it.

Shairplay Sync will automatically choose a suitable latency for iTunes and for AirPlay devices such as the AppleTV.

What else?
--------------
* Shairport Sync offers finer control at very top and very bottom of the volume range. See http://tangentsoft.net/audio/atten.html for a good discussion of audio "attenuators", upon which volume control in Shairport Sync is modelled. See also the diagram of the volume transfer function in the documents folder.
* Shairport Sync will mute properly if the hardware supports it.
* If it has to use software volume and mute controls, the response time is shorter than before -- now it responds in 0.15 seconds.
* Shairport Sync sends back a "busy" signal if it's already playing audio from another source, so other sources can't "barge in" on an existing Shairport Sync session. (If a source disappears without warning, the session automatically terminates after two minutes and the device becomes available again.)

Status
------
Shairport Sync is working on Raspberry Pi with Raspian and OpenWrt, and it runs on a Linksys NSLU2 using OpenWrt. It also works on a standard Ubuntu laptop. It works well with built-in audio and with a variety of USB-connected Amplifiers and DACs, including a cheapo USB "3D Sound" dongle, a first generation iMic and a Topping TP30 amplifier with a USB DAC input. It also works with the IQAudIO Pi-DAC on the latest version of Raspian -- please see http://iqaudio.com/wp-content/uploads/2014/06/IQaudIO_Doc.pdf for details.

Shairport Sync compiles and runs pretty well on the built-in sound card of a Raspberry Pi model B under Raspian. Due to the limitations of the sound card, you wouldn't mistake the output for HiFi, but it's really not too shabby.
USB-connected sound cards work well on the latest version of Raspian. Older versions of Raspian a known problem -- see http://www.raspberrypi.org/forums/viewtopic.php?t=23544, so it is wise to update.
Shairport Sync runs well on the same hardware using OpenWrt.

Shairport Sync runs on Ubuntu and Debian 7 inside VMWare Fusion 6.0.3 on a Mac, but synchronisation does not work -- possibly because the soundcard is being emulated.

Shairport Sync works only with the ALSA back end. You can try compiling the other back ends in as you wish, but it definitely will not work properly with them. Maybe someday...

One other difference from other versions of Shairport is that the Shairport Sync build process now uses GNU autotools and libtool to examine and configure the build environment -- very important for cross compilation. All previous versions looked in the current system to determine which packages were available, instead of looking at what packages were available in the target system.

Building And Installing
---------------------
If you're interested in Shairport Sync for OpenWrt, there's an OpenWrt package at https://github.com/mikebrady/shairport. OpenWrt doesn't support the IQaudIO Pi-DAC.

Otherwise, follow these instructions.

The following libraries are required:
* OpenSSL or PolarSSL
* Avahi
* ALSA
* libdaemon
* autoconf
* libtool

Optional
* libsoxr

Many linux distributions have Avahi and OpenSSL already in place, so normally it probably makes sense to choose those options rather than tinysvcmdns or PolarSSL. Libsoxr is available in recent linux distributions and it requires lots of horsepower � chances are an embedded processor won't be able to keep up.

Debian, Ubuntu and Raspian users can get the basics with:

`apt-get install autoconf libtool libdaemon-dev libasound2-dev`
`apt-get install avahi-daemon libavahi-client-dev` if you want to use Avahi (recommended), or use `--with-tinysvcmdns` otherwise.
`apt-get install libopenssl-dev` if you want to use OpenSSL and libcrypto, or use PolarSSL otherwise.
`apt-get install libpolarssl-dev` if you want to use PolarSSL, or use OpenSSL/libcrypto otherwise.
`apt-get install libsoxr-dev` if you want support for libsoxr-based resampling. Not yet part of  Raspian.

Download Shairport Sync:

`git clone -b 2.1 https://github.com/mikebrady/shairport-sync.git`

Next, `cd` into the shairport-sync directory and execute the following commands:

`$ autoreconf -i -f`

Choose the appropriate `--with-*` options:

`--with-alsa` for the ALSA audio back end. This is required.
`--with-avahi` or `--with-tinysvcmdns` for mdns support support.
`--with-openssl`  or `--with-polarssl` for encryption and related utilities.
`--with-soxr` for libsoxr-based resampling.

Here is an example:

`$ ./configure --with-alsa --with-avahi --with-openssl --with-soxr`

`$ make` 

Run `$sudo make install` to install `shairport-sync` along with an initscript which will automatically launch it at startup. The settings in the init script are the most basic defaults, so you will want to edit it -- the file `/etc/init.d/shairport-sync` -- to give the service a name, use a different card, use the hardware mixer and volume control, etc. -- there are some examples in the script file.

Configuring Shairport Sync
--------
Shairport Sync can mostly look after itself, so when you first install it, a default configuration is used which should work in almost any system with a sound card. If there is a problem, it will be noted in the logfile, typically `/etc/log/syslog`. However, to get the most out of your software and hardware, you need to adjust some of the settings. To understand what follows, note that settings and parameters are passed to Shairport Sync through command line arguments. The purpose of the init script at /etc/init.d/shairport-sync is to launch or terminate Shairport Sync while passing the correct arguments to it. You are perfectly free to remove the init file and launch and terminate Shairport Sync yourself directly; indeed it is useful when you are troubleshooting the program. If you do launch it directly, make sure it isn't running already!

Don't forget you can launch Shairport Sync with the -h option to get some help on the options available.

These are the main options:

The `-a` option allows you to specify the name Shairport Sync will appear as on the network. If you don't specify a name, the name Shairport Sync on <your computer name>" will be chosen.

The `-S` option allows you to specify the kind of "stuffing" or interpolation to be used � `basic` (default) for simple insertion/deletion  or `soxr` for smoother resampling-based interpolation.

These may be also of interest:

The `-B`, `-E` and `-w` options allow you to specify a program to execute before (`-B`) and after (`-E`) Shairport Sync plays. This is to facilitate situations where something has to be done before and  after playing, e.g. switching on an amplifier beforehand and switching it off afterwards. Use the `-w` option for Shairport Sync to wait until the respective commands have been completed before continuing. Please note that the full path to the programs must be specified, and script files will not be executed unless they are marked as executable and have the standard `#!/bin/...` first line. (This behaviour may be different from other Shairports.)

The `-V` option gives you version information about  Shairport Sync and then quits.

The `-k` option causes Shairport Sync to kill an existing Shairport Sync daemon and then quit. You need to have sudo privileges for this.

The `-v` option causes Shairport Sync to print some information and debug messages.

The `d` option causes Shairport Sync to properly daemonise itself, going into the background. You may need sudo privileges for this.

Apart from arguments of Shairport Sync, there are also arguments for controlling the ALSA audio library. ALSA arguments follow a `�` on the command line.

These are important because her you specify the actual audio device you wish to use and you give Shairport Sync important information about the capabilities of the device. The important ALSA arguments are:

The `-d` option which allows you to specify the audio device to use. Typical values would be `default` (default), `hw:0`, `hw:1`, etc. The importance of this setting is obvious.

The following two settings are very important for maximum performance. If your audio device has a hardware mixer and volume control, then Shairport Sync can use it to give faster response to volume and mute control and it can offload some work from the processor, potentially allowing a slower processor to work.

The `-t` option allows you to specify the type of audio mixer � `software` (default) or `hardware`.

The `-c` option allows you to specify the name of the volume control on the hardware mixer.

The init script at /etc/init.d/shairport-sync has a bare minimum set of options (see line 60):

`-d`

Basically all it does is put the program in the background ("daemon") mode, selects the default output device and uses software volume control. You should modify it, if possible, to get hardware volume control. Here are some examples of complete commands. If you are modifying the init script, you don't need the `shairport-sync` at the start, but include the `-d` options, as it puts the program into daemon mode. There are some commented-out examples in the init script; see lines 61�63.

This give the service a particular name � "Joe's Stereo" and specifies that audio device hw:0 be used.
`shairport-sync -d -a "Joe's Stereo" -- -d hw:0`

For best results � including getting true mute and instant response to volume control and pause commands � you should access the hardware volume controls. Use `amixer` or `alsamixer` or similar to discover the name of the volume controller to be used after the `-c` option.

Here is an example for for a Raspberry Pi using its internal soundcard � device hw:0 � that drives the headphone jack:

`shairport-sync -d -a "Mike's Boombox" -- -d hw:0 -t hardware -c PCM`

Here is an example of using soxr-based resampling and driving a Topping TP30 Digital Amplifier, which has an integrated USB DAC and which is connected as audio device `hw:1`:

`shairport-sync -d -a Kitchen -S soxr -- -d hw:1 -t hardware -c PCM`

For a cheapo "3D Sound" USB card (Stereo output and input only) on a Raspberry Pi:

`shairport-sync -d -a "Front Room" -- -d hw:1 -t hardware -c Speaker`

For a first generation Griffin iMic on a Raspberry Pi:

`shairport-sync -d -a "Attic" -- -d hw:1 -t hardware -c PCM`

For an NSLU2, which has no internal soundcard, there appears to be a bug in ALSA -- you can not specify a device other than "default". Thus:

On an NSLU2, to drive a first generation Griffin iMic:

`shairport-sync -d -a "Den" -- -t hardware -c PCM`

On an NSLU2, to drive the "3D Sound" USB card:

`shairport-sync -d -a "TV Room" -- -t hardware -c Speaker`

Latency
-------
Latency is the exact time from a sound signal's original timestamp until that signal actually "appears" on the output of the DAC, irrespective of any internal delays, processing times, etc. in the computer. From listening tests, it seems that there are two latencies in current use:
* If the source is iTunes, a latency of 99,400 frames seems to bring Shairport Sync into exact synchronisation both with the speakers on the iTunes computer itself and with AirPort Express receivers.
* If the source is an AirPlay device, the latency seems to be exactly 88,200 frames. AirPlay devices include AppleTV, iPod, iPad and iPhone and Quicktime Player on Mac.

Shairport Sync therefore has default latencies for iTunes and AirPlay sources: 99,400 frames for iTunes and 88,200 for all AirPlay devices.

You can set your own iTunes latency with the `-i` or `--iTunesLatency` option (e.g. `-i 99400` or `--iTunesLatency=99400`). Similarly you can set an AirPlay latency with the `-A` or `--AirPlayLatency` option. If you set a latency with `-L` (deprecated) it overrides both the AirPlay and iTunes latency values. Basically, you shouldn't use it except for experimentation.

Resynchronisation
-------------
Shairport Sync actively maintains synchronisation with the source. 
If synchronisation is lost -- say due to a busy source or a congested network -- Shairport Sync will mute its output and resynchronise. The loss-of-sync threshold is a very conservative 50 ms -- i.e. the actual time and the expected time must differ by more than 50 ms to trigger a resynchronisation. Smaller disparities are corrected by insertions or deletions, as described above.
You can vary the resync threshold, or turn resync off completely, with the `-r` opton.

Some Statistics
---------------
If you run Shairport Sync from the command line without daemonising it (omit the `-d`), and if you turn on one level of verbosity (include `-v`), e.g. as follows for the Raspberry Pi with "3D Sound" card:

`shairport-sync -a "Shairport Sync" -v -- -d hw:1 -t hardware -c Speaker`

it will print statistics like this occasionally:

`Sync error: -35.4 (frames); net correction: 24.2 (ppm); corrections: 24.2 (ppm); missing packets 0; late packets 5; too late packets 0; resend requests 6; min DAC queue size 4430.`

"Sync error" is the average deviations from exact synchronisation. The example above indicates that the output is on average 35.4 frames ahead of exact sync. Sync is allowed to wander by � 88 frames (� 2 milliseconds) before a correction will be made.

"Net correction" is actually the net sum of corrections -- the number of frame insertions less the number of frame deletions -- given as a moving average in parts per million. After an initial settling period, it represents the divergence between the source clock and the sound device's clock. The example above indicates that the output DAC's clock is running 24.2 ppm faster than the source's clock.

"Corrections" is the number of frame insertions plus the number of frame deletions (i.e. the total number of corrections), given as a moving average in parts per million. The closer this is to the absolute value of the drift, the fewer "unnecessary" corrections that are being made.

For reference, a drift of one second per day is approximately 11.57 ppm. Left uncorrected, even a drift this small between two audio outputs will be audible after a short time. The above sample is from a second-generation iPod driving a Raspberry Pi which is connected over Ethernet.

It's not unusual to have resend requests, late packets and even missing packets if some part of the connection to the Shairport Sync device is over WiFi. Sometimes late packets can be asked for and received multiple times. Sometimes late packets are sent and arrive too late, but have already been sent, so weren't needed anyway...

"min DAC queue size" is the minimum size the queue of samples in the output device's hardware buffer was measured at. It is meant to stand at 0.15 seconds = 6,615 samples, and will go low if the processor is very busy. If it goes below about 2,000 then it's a sign that the processor can't really keep up.
