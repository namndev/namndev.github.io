---
layout: post
title: Creating A Production Ready Multi Bitrate HLS VOD stream
# subtitle: ... or not to be?
tags: [FFMPEG, HLS, Codec]
---

[`HLS`](https://en.wikipedia.org/wiki/HTTP_Live_Streaming) is one of the most prominent video streaming formats on desktop and mobile browsers. Since end users have different screen sizes and different network performance, we want to create multiple renditions of the video with different resolutions and bitrates that can be switched seamlessly, this concept is called MBR (Multi Bit Rate).

For this task we will use [`ffmpeg`](https://en.wikipedia.org/wiki/FFmpeg), a powerful tool that supports conversion of various video formats from one to another, including HLS both as input and output.

In this guide will show a real world use of `ffmpeg` to create MBR HLS VOD stream from a static input file.

## Installing FFMPEG

`ffmpeg` is a cross platform program that can run on Windows and OS X as well as Linux.

### Windows

- Download latest version from [here](http://ffmpeg.zeranoe.com/builds/)
- Unzip the archive to a folder
- Open a command prompt in the unzipped folder
- Run `./ffmpeg` - you should see ffmpeg version and build information

### OS X

- Install [homebrew](https://brew.sh/)
- Run `brew install ffmpeg` (extra options can be seen by running brew options ffmpeg)
- Run `./ffmpeg` - you should see ffmpeg version and build information

### Ubuntu

```bash
sudo add-apt-repository ppa:mc3man/trusty-media  
sudo apt-get update  
sudo apt-get install -y ffmpeg
```

### CentOS / Fedora

```bash
yum install -y ffmpeg
```

Latest binaries for all platforms, source code and more information is available at [ffmpeg’s official website](http://ffmpeg.org/download.html)

### Source Media

I will use a `.mkv` file `beach.mkv`, though `ffmpeg` will consume most of the common video formats. sample files can be downloaded from [here](http://www.sample-videos.com/) and [here](http://www.h264info.com/clips.html).

To inspect the file properties run the following command:

```bash
ffprobe -hide_banner beach.mkv
```

The file is identified as mkv, 21 seconds long, overall bitrate 19264 kbps, containing one video stream of 1920x1080 23.98fps in h264 codec, and one AC3 audio stream 48kHz 640 kbps.

## Multi Bitrate Conversion

### First rendition

Lets build a command for one rendition:

```bash
ffmpeg -i beach.mkv -vf scale=w=1280:h=720:force_original_aspect_ratio=decrease \
-c:a aac -ar 48000 -b:a 128k -c:v h264 -profile:v main -crf 20 \
-g 48 -keyint_min 48 -sc_threshold 0 -b:v 2500k -maxrate 2675k \
-bufsize 3750k -hls_time 4 -hls_playlist_type vod \
-hls_segment_filename beach/720p_%03d.ts beach/720p.m3u8
```


- `-i beach.mkv` - set `beach.mkv` as input file
- `-vf "scale=w=1280:h=720:force_original_aspect_ratio=decrease"` - scale video to maximum possible within 1280x720 while preserving aspect ratio
- `-c:a aac -ar 48000 -b:a 128k` - set audio codec to AAC with sampling of 48kHz and bitrate of 128k
- `-c:v h264` - set video codec to be H264 which is the standard codec of HLS segments
- `-profile:v main` - set H264 profile to main - this means support in modern devices - [read more](http://blog.mediacoderhq.com/h264-profiles-and-levels/)
- `-crf 20` - Constant Rate Factor, high level factor for overall quality
- `-g 48 -keyint_min 48` - ***IMPORTANT*** create key frame (I-frame) every 48 frames (~2 seconds) - will later affect correct slicing of segments and alignment of renditions
- `-sc_threshold 0` - don't create key frames on scene change - only according to `-g`
- `-b:v 2500k -maxrate 2675k -bufsize 3750k` - limit video bitrate, these are rendition specific and depends on your content type - [read more](https://docs.peer5.com/guides/production-ready-hls-vod/#how-to-choose-the-right-bitrate)
- `-hls_time 4` - segment target duration in seconds - the actual length is constrained by key frames
- `-hls_playlist_type vod` - adds the #EXT-X-PLAYLIST-TYPE:VOD tag and keeps all segments in the playlist
- `-hls_segment_filename beach/720p_%03d.ts` - explicitly define segments files names
- `beach/720p.m3u8` - path of the playlist file - also tells ffmpeg to output HLS (`.m3u8`)
This will generate a VOD HLS playlist and segments in `beach` folder.

### Multiple renditions

Each rendition requires its own parameters, though `ffmpeg` supports multiple inputs and outputs so all the renditions can be generated in parallel with one long command.

It's very important that besides the resolution and bitrate parameters the commands will be identical so that the renditions will be properly aligned, meaning key frames will be set in the exact same positions to allow smooth switching between them on the fly.

We will create 4 renditions with common resolutions:

- `1080p` 1920x1080 _(original)_
- `720p` 1280x720
- `480p` 842x480
- `360p` 640x360

```bash
ffmpeg -hide_banner -y -i beach.mkv \
  -vf scale=w=640:h=360:force_original_aspect_ratio=decrease -c:a aac -ar 48000 -c:v h264 -profile:v main -crf 20 -sc_threshold 0 -g 48 -keyint_min 48 -hls_time 4 -hls_playlist_type vod  -b:v 800k -maxrate 856k -bufsize 1200k -b:a 96k -hls_segment_filename beach/360p_%03d.ts beach/360p.m3u8 \
  -vf scale=w=842:h=480:force_original_aspect_ratio=decrease -c:a aac -ar 48000 -c:v h264 -profile:v main -crf 20 -sc_threshold 0 -g 48 -keyint_min 48 -hls_time 4 -hls_playlist_type vod -b:v 1400k -maxrate 1498k -bufsize 2100k -b:a 128k -hls_segment_filename beach/480p_%03d.ts beach/480p.m3u8 \
  -vf scale=w=1280:h=720:force_original_aspect_ratio=decrease -c:a aac -ar 48000 -c:v h264 -profile:v main -crf 20 -sc_threshold 0 -g 48 -keyint_min 48 -hls_time 4 -hls_playlist_type vod -b:v 2800k -maxrate 2996k -bufsize 4200k -b:a 128k -hls_segment_filename beach/720p_%03d.ts beach/720p.m3u8 \
  -vf scale=w=1920:h=1080:force_original_aspect_ratio=decrease -c:a aac -ar 48000 -c:v h264 -profile:v main -crf 20 -sc_threshold 0 -g 48 -keyint_min 48 -hls_time 4 -hls_playlist_type vod -b:v 5000k -maxrate 5350k -bufsize 7500k -b:a 192k -hls_segment_filename beach/1080p_%03d.ts beach/1080p.m3u8
```
### Master playlist

The HLS player needs to know that there are multiple renditions of our video, so we create an HLS master playlist to point them and save it along side the other playlists and segments.

***playlist.m3u8***

```bash

#EXTM3U
#EXT-X-VERSION:3
#EXT-X-STREAM-INF:BANDWIDTH=800000,RESOLUTION=640x360
360p.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=1400000,RESOLUTION=842x480
480p.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=2800000,RESOLUTION=1280x720
720p.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=5000000,RESOLUTION=1920x1080
1080p.m3u8
```

### Example Conversion Script

Here is an example conversion script [create-hls-vod.sh](https://gist.github.com/namndev/26f90f2f591886bedf1047c4398a98b5)

Running:

```bash
bash create-vod-hls.sh beach.mkv
```

will produce:

```bash
    beach/
      |- playlist.m3u8
      |- 360p.m3u8
      |- 360p_001.ts
      |- 360p_002.ts
      |- 480p.m3u8
      |- 480p_001.ts
      |- 480p_002.ts
      |- 720p.m3u8
      |- 720p_001.ts
      |- 720p_002.ts
      |- 1080p.m3u8
      |- 1080p_001.ts
      |- 1080p_002.ts  
```

## FAQ

### How to choose the right bitrate

Bitrate is dependant mostly on the resolution and the content type. When setting bitrate too low an image pixelization will occur especially in areas where there is rapid movement, when bitrate is too high the output files might be excessively big without additional value.

To choose the right bitrate one must understand his type of content. Content with high motion such as sports or news events will require higher bitrate to avoid pixelization while low motion content such as music concerts and interviews will suffice lower bitrate without apparent changes to quality.

Here are some good defaults to start from:

| Quality |	Resolution |	bitrate - low motion | bitrate - high motion | audio bitrate |
|:------------- |:---------------:|:-------------:|:---------------:|:-------------:|
| 240p |	426x240	| 400k | 	600k |	64k |
| 360p	 | 640x360	| 700k	 | 900k | 96k |
| 480p	 | 854x480	| 1250k	| 1600k	| 128k |
| HD 720p	| 1280x720 |	2500k	| 3200k |	128k |
| HD 720p 60fps |	1280x720 |	3500k	|4400k |	128k |
|Full HD 1080p	| 1920x1080	| 4500k	| 5300k |	192k |
|Full HD 1080p 60fps |	1920x1080 |	5800k |	7400k	|192k |
|4k	 |3840x2160	| 14000k	| 18200	| 192k |
|4k 60fps	| 3840x2160	| 23000k	| 29500k |	192k|


### How do I feed ffmpeg through stdin?

`ffmpeg` has a special `pipe`: flag that instruct ffmpeg to consume stdin as media.

```bash
cat clip.mp4 | ffmpeg -f mp4 -i pipe: output.avi
```

### What’s the difference between avconv and ffmpeg?

`avconv` is a fork (clone) of `ffmpeg` that was created by a group of developers of ffmpeg due to project management issues. While both are actively maintained, its recommended to use ffmpeg since it has larger community as explained [here](https://wiki.debian.org/Debate/libav-provider/ffmpeg)