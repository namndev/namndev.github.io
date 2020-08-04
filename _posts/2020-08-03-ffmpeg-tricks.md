---
layout: post
title: "FFmpeg Tricks"
tags: [Video, Codec]
comments: true
---

# 1. Convert audio

I back my CDs up to FLAC and then transcode to AAC (.m4a) files for portability on my iPhone and the wife's android phone.

```bash
ffmpeg -i my_song.flac -vn -c:a libfdk_aac -b:a 500k my_song.m4a
```

- `-vn`: skip video stream.

To excute for many flac in a folder:

```bash
find . -name '*.flac' -exec sh -c 'ffmpeg -i "$1" -vn -c:a libfdk_aac -b:a 500k "${1%.flac}.m4a"' _ {} \;
```

> Note: you need build `libfdk_aac` for ffmpeg.

## 2. Add text subtitles

In a folder video that has many files: mp4, ass, srt ... I want to merge subtile file to video. 

```bash
ffmpeg -i my_video.mp4 -i my_video.srt -map 0:0 -map 0:1 -map 1:0 -c:v copy -c:a copy -c:s mov_text -metadata:s:s:0 language=eng my_video_sub.mp4
```

- `-metadata:s:s:0 language=eng` : set language title for subtitle.
- `-disposition:s:s:0 forced`: set this language is default.

To excute for many video with srt in a folder:

```bash
find . -name '*.mp4' -exec sh -c 'ffmpeg -i "$1" -i "${1%.mp4}.srt" -map 0:0 -map 0:1 -map 1:0 -c:v copy -c:a copy -c:s mov_text -metadata:s:s:0 language=eng "${1%.mp4}_sub.mp4"' _ {} \;
```

If you have a file `.ass`, you can convert to `srt`

```bash
ffmpeg -i subtitles.ass subtitles.srt
```