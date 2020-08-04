---
layout: post
title: "FFmpeg Tricks"
tags: [Video, Codec]
comments: true
---

# 1. Audio Conversion

Say you have the audio file named my_audio.wav and want to convert it in a mp3.

```bash
ffmpeg -i my_audio.wav my_audio.mp3
```
- `-i` is input file

Really easy, it isn’t ? change the extension of the output file with any supported format to get different formats.

Now, I back my CDs up to FLAC and then transcode to AAC (.m4a) files for portability on my iPhone and the wife's android phone.

```bash
ffmpeg -i my_song.flac -vn -c:a libfdk_aac -b:a 500k my_song.m4a
```

- `-vn`: skip video stream.

To excute for many flac in a folder:

```bash
find . -name '*.flac' -exec sh -c 'ffmpeg -i "$1" -vn -c:a libfdk_aac -b:a 500k "${1%.flac}.m4a"' _ {} \;
```

> Note: you need build `libfdk_aac` for ffmpeg.

# 2 Video Conversion

The basic usage it’s like the example saw for the audio so you can simply write:

```bash
ffmpeg -i my_video.mpeg -s 500×500 my_video.mkv
```

- `-s`: `size` set video resolution size (Width x Height)

This will convert my_video.mpeg file to my_video.mkv and will resize the video resolution to 500×500

# 3. Add text subtitles

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

# 4. Extract images from a video

Sometimes is useful to extract some images from a movie, and ffmpeg can do this easily:

```bash
ffmpeg-i my_video.mp4 my_video.mpeg
```

This will create 25 images for every 1 second, but it may serve us to have more or less images, this can be achieved with the parameter `-r`

- `-r` fps Set frame rate (default 25)

```bash
ffmpeg -i my_video.mpeg -s 500×500 my_video.mp4
```

With this command you’ll get 1 image for every second.

Start e duration

You can also give a start time and the duration with the flags:

- `-ss` position Seek to given time position in seconds. `hh:mm:ss[.xxx]` syntax is also supported.

- `-t` duration Restrict the transcoded/captured video sequence to the duration specified in seconds. `hh:mm:ss[.xxx]` syntax is also supported.

This command will take 25 images images every second beginning at the tenth second, and continuing for 5 seconds

```bash
ffmpeg -i test.mpg image%d.jpg
```

# 5. Extract audio from a video
With `ffmpeg` you can also mix video and audio, so we can extract an mp3 track from a video:

```bash
ffmpeg -i test.mpg -r 1 image%d.jpg
```

In this example we have used the flag `-f`.

- `-f`: `fmt` Force the format.

To get the same result it’s also possible to use the option to disable video capture:

- `-vn` Disable video recording

```bash
ffmpeg -i test.mpg -r 25 -ss 00:00:10 -t 00:00:05 images%05d.png
```

# 6. Create a screencast
With ffmpeg it’s also possible to create a simple screencast, capturing your desktop.

To do this we’ll use some of the flags show in the former example:

```bash
ffmpeg -i video.avi -f mp3 audio.mp3
```

Note: 0.0 is display.screen number of your X11 server, same as the DISPLAY environment variable.

This will save 25 frame per second of your wxga screen (or you can use `-s` with a resolution like `-s` 1024×768) and put them in a mpg video in /tmp.

# 7. Create a video from images
Say you have a lot of images named `img001.jpg`, `img002.jpg` and so on in sequence, you can convert them into a movie with this command:

```bash
ffmpeg -r 1/5 -start_number 2 -i img%03d.png -c:v libx264 -r 30 -pix_fmt yuv420p out.mp4
```

This line worked fine but I want to create a video file from images in another folder. Image names in my folder are:

```bash
img001.jpg
img002.jpg
img003.jpg
...
```

[Read more](https://stackoverflow.com/questions/24961127/how-to-create-a-video-from-images-with-ffmpeg)

# 8. Capture a video from the webcam
To record video run ffmpeg with arguments such as these:

```bash
ffmpeg -f x11grab -r 25 -s wxga -i :0.0 /tmp/outputFile.mpg
```

To record both audio and video use:

```bash
ffmpeg -f image2 -i img%d.jpg /tmp/a.mpg
```

These are just some examples, ffmpeg can do a lot in editing audio and video, and in the net there are a lot of examples about it.