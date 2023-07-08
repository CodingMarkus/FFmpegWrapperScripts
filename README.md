# FFmpeg Wrapper Scripts

## Help as shown by `enc.video`

```
Syntax:

    ./enc.video [scale=<w:h|x%|keyword>] [volume=<p%|max>]
        <profile> <input> <output>

    (order of scale and volume is irrelevant)


Valid profile names are:

    vlow  - Very low quality (H.264/AAC).
    low   - Low quality (H.264/AAC).
    std   - Standard quality (H.264/AAC).
    high  - High quality (H.264/AAC).
    vhigh - Very high quality (H.264/AAC).

	500k - Limit total bandwidth to 500kbit/s (H.264/AAC).
	1m   - Limit total bandwidth to 1Mbit/s (H.264/AAC).
	2m   - Limit total bandwidth to 2Mbit/s (H.264/AAC).
	3m   - Limit total bandwidth to 2Mbit/s (H.264/AAC).
	4m   - Limit total bandwidth to 4Mbit/s (H.264/AAC).
	6m   - Limit total bandwidth to 6Mbit/s (H.264/AAC).
	8m   - Limit total bandwidth to 8Mbit/s (H.264/AAC).

    scrcast - Screencast with very low bitrate (H.264/AAC).
    webcast - Web screencast with low bitrate (VP9/Opus).

    bug     - Screen recording for bug reporting (H.264).
	bugwebm - As bug but using VP9 (VP9).

    gif  - Animated GIF (or APNG).
    gifo - Animated GIF (or APNG) w/ optimized palette.

    transform - Copy audio/video data unmodified, just rewrite container.


Scale can either provide both, width (w) and height (h) or only one of both by
setting the other one to -1 in which case the aspect ratio is kept.
Alternatively, the following special keyword values are supported:

    half    - Halves the video resolution.
    quarter - Quarters the video resolution.
    double  - Doubles the video resolution.
    x%      - Scales the video by x percent (up or down).


Volume can either be in percent, allowing increasing and decreasing volume by
any amount desired (up to the point where the output clips), or it can be
"max" in which case the maximum amplication is calculated that is possible
without causing any clipping.


The container format is guessed by the file extension of the output file. E.g.
".mp4" generates an MPEG 4 container file, whereas ".mkv" would generate a
Matroska container, and ".webm" a WebM container. Similar using ".apng"
with the GIF profile creates an animated PNG file. Note that not all codecs can
be used in all container formats.

If output is a folder, the same file name as input will be used, except for
the file extension which will be the default extension for the default
container format of the chosen profile.
```

# Help as shown by `enc.audio` Script

```
Syntax:

    enc.audio [options] <profile> <input_file> <output_file>

    enc.audio [options] <profile> <input_file> ... <output_directory>


The container format is guessed by the file extension of the output file.
E.g. ".mp4" generates an MPEG 4 container file, whereas ".mp3" would
generate a MP3 container, and ".aac" a raw AAC data stream.

If output is a directory, the same file name as input will be used, except for
the file extension which will be the default extension for the default
container format of the chosen profile, unless overridden by the "extension"
option.

NOTE:
Not all codecs can be used in all container formats!


Options:

    NOTE:
    Options can be shortened. E.g. "vol" and "volume" are the same option.

    vol[ume]=<p%|max>

        Volume can either be in percent ("90%"), allowing increasing and
        decreasing volume by any amount desired (up to the point where the
        output clips), or it can be "max" in which case the maximum
        amplication is calculated that is possible without causing any
        clipping.


    ext[ension]=<ext>

        Overrides output file extension in case output is a directory,
        e.g. "ext=mp3". Has no effect if output is a file name.


Valid profile names are:

    mp3poor - Variable bitrate MP3, poor quality.
    mp3low  - Variable bitrate MP3, low quality.
    mp3std  - Variable bitrate MP3, standard quality.
    mp3high - Variable bitrate MP3, high quality.

    aacpoor - Variable bitrate AAC, poor quality.
    aaclow  - Variable bitrate AAC, low quality.
    aacstd  - Variable bitrate AAC, standard quality.
    aachigh - Variable bitrate AAC, high quality.

    archive  - FLAC lossless compression, suitable for archiving music.

    webcast       - Webcast with low bitrate (Vorbis mono).
    webcasthigh   - Webcast with high bitrate (Vorbis mono).
    webcasthq     - Webcast with low bitrate, better quality (Opus mono).
    webcasthqhigh - Webcast with high bitrate, better quality (Opus mono).

    transform - Copy audio data unmodified, just rewrite container.

    fixaac - Fix broken AAC streams with bad timing or bad average bitrate
             that transform cannot fix. All meta information will be lost!
```

## Notes about FFmpeg AAC

There are three AAC codecs available for FFmpeg:

- Fraunhofer FDK AAC (`libfdk_aac`): \
The Fraunhofer FDK AAC codec library. This encoder has the highest quality and good speed. Additional to the LC profile also supports the HE and the HEv2 AAC profile for encoding at very low bitrate. Is also supports VBR encoding.

- Audio Toolbox AAC (`aac_at`): \
This codec uses Apple's AAC library and is only available on macOS systems. The quality is almost identical to the one of the Fraunhofer FDK AAC, maybeslightly worse at very low bitrates, therefor the encoder is lightning fast (up to twice the speed of the Fraunhofer's one). It also supports the LC, the HE and the HEv2 profile, as well as VBR encoding.

- Native FFmpeg AAC Encoder (`aac`): \
This is the worst encoder of the three. It's only about half the speed of the Fraunhofer FDK AAC and it's quality is definitely worse, quite noticeable at very low bitrates. Further it only supports the LC profile and while it has a VBR mode, you don't want to use it as it produces worse results than a CBR encoding with even lower bitrate does.

Both scripts support all three encoders and will always favor Audio Toolbox AAC if available, as it offers high quality at very high speed. It will fall back to Fraunhofer FDK AAC if available, as it offers the best quality with still decent speed. Only if it has to, it will use the Native FFmpeg AAC Encoder, because it produces okay results if the bitrate is high enough and no VBR encoding is being used but otherwise cannot be recommended.

Only in situations where HE or HEv2 encoding is required, as otherwise the result is just crap, it will refuse to use the Native FFmpeg AAC Encoder as that encoder has no support for these profiles and encoding using LC profile at these bitrates produces horrible results.

## Notes about FFmpeg for macOS

The best way to get FFmpeg for macOS is using [MacPorts](https://www.macports.org/). Just install it using the installer packet for your macOS version, open a terminal window and run:

```
sudo port selfupdate && sudo port upgrade ffmpeg +gpl2 +gpl3 +nonfree
```
The `+...` syntax activates variants. Only when installed as shown above you will get codecs and optimizations that use GPL2, GPL3, as well as those using any kind of non-free license. Otherwise you only get codecs that use LGPL2 license.

The reason why you should favor MacPorts over other packet manages like Brew (☠️) is that MacPorts isolates itself as much as possible from the rest of the system by putting almost all files under `/opt/local` and installs all binary files and libraries in such a way, that only a process with root access an modify them (they all belong to the `root` user, just like all standard system binaries and libraries that Apple ships with macOS). This also ensures that multiple admin users on a macOS system can all share a single installation of MacPorts without any issues.

Brew on the other hand installs its files to `/usr/local/Cellar` (very bad directory choice), symlinks binaries to `/usr/local/bin` which can cause all kind conflicts with manual installed software and some third party apps, and the files are owned by the user that installed Brew, which has two negative effects: First of all you will run into issues if another admin user tries to use or manage Brew as well on the same system. Yet even worse, this is a huge security problem! 💣 Every app/process that the user runs, who installed Brew, can modify the Brew binaries without requiring root access rights, so it's easy for malware to replace your binaries there with malware code. And now consider that you ever run a Brew binary with `sudo`, as you want to access a file that your user cannot otherwise access? If that binary was replaced by maleware, it just got root access and will take over your entire system!

**Don't use Brew if you care for the security of your system!!!**