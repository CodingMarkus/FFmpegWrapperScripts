# FFmpeg Wrapper Scripts

## Help as shown by `enc_video`

```

Syntax:

    ./enc_video [scale=<w:h|x%|keyword>] [volume=<p%|max>]
        <profile> <input> <output>

    (order of scale and volume is irrelevant)


Valid profile names are:

    poor  - Poor quality (H.264/AAC).
    vlow  - Very low quality (H.264/AAC).
    low   - Low quality (H.264/AAC).
    std   - Standard quality (H.264/AAC).
    high  - High quality (H.264/AAC).
    vhigh - Very high quality (H.264/AAC).

    tiny   - Tiny file size, < 1 MiByte/min (H.264/AAC).
    vsmall - Very small file size, < 2 MiByte/min (H.264/AAC).
    small  - Small file size, < 4 MiByte/min (H.264/AAC).
    medium - Medium file size, < 8 MiByte/min (H.264/AAC).
    big    - Big file size, < 12 MiByte/min (H.264/AAC).
    vbig   - Very big file size, < 16 MiByte/min (H.264/AAC).

    scrcast   - Screencast with very low bitrate (H.264/AAC).
    scrcasthq - Screencast with very low bitrate (H.265/AAC).
    webcast   - Web screencast with low bitrate (VP9/Vorbis).

    bug    - Screen recording for bug reporting (H.264).
    bugm   - As bug but if lots of motion is visible (H.264).
    bughq  - Screen recording for bug reporting (H.265).
    bughqm - As bughq but if lots of motion is visible (H.265).

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
```

# Help as shown by `enc_audio` Script

```
Syntax:

    ./enc_audio [volume=<p%|max>] <profile> <input> <output>


Valid profile names are:

    mp3poor - Variable bitrate MP3, poor quality.
    mp3low  - Variable bitrate MP3, low quality.
    mp3std  - Variable bitrate MP3, standard quality.
    mp3high - Variable bitrate MP3, high quality.

    aacpoor - Variable bitrate AAC, poor quality.
    aaclow  - Variable bitrate AAC, low quality.
    aacstd  - Variable bitrate AAC, standard quality.
    aachigh - Variable bitrate AAC, high quality.

    aacarc   - AAC with quality good enough for archiving music.
    archive  - FLAC lossless compression, suitable for archiving music.

    speechlow  - Speech only, low quality (AAC HE mono).
    speechstd  - Speech only, standard quality (AAC HE mono).
    speechhigh - Speech only, high quality (AAC HE mono).

    webcast       - Webcast with low bitrate (Vorbis mono).
    webcasthigh   - Webcast with higher bitrate (Vorbis mono).
    webcasthq     - Webcast with low bitrate (Opus mono).
    webcasthqhigh - Webcast with higher bitrate (Opus mono).

    transform - Copy audio data unmodified, just rewrite container.

    fixaac - Fix broken AAC streams with bad timing or bad average bitrate
             that transform cannot fix. All meta information will be lost!


Volume can either be in percent, allowing increasing and decreasing volume by
any amount desired (up to the point where the output clips), or it can be
"max" in which case the maximum amplication is calculated that is possible
without causing any clipping.


The container format is guessed by the file extension of the output file. E.g.
".mp4" generates an MPEG 4 container file, whereas ".mp3" would generate a
MP3 container, and ".aac" a raw AAC data stream. Note that not all codecs can
be used in all container formats.
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