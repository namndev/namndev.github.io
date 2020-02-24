---
layout: post
title: "Compile FFmpeg with RTMPS for Facebook"
image: https://clipartart.com/images/facebook-live-logo-clipart.jpg
tags: [Video, Live, Facebook, Codec, HLS]
comments: true
---

Facebook announced that "Live Video streams on __Pages and Workplace__ can use the non-encrypted standard RTMP protocol until __November 1st, 2019__, after which RTMP will no longer be supported".

On __May 1st, 2019__, the Real-time Messaging Protocol (RTMP) has been   deprecated from the Live API, GoLive Dialog, and Publisher Pages. RTMPS (RTMP over a TLS/SSL connection) will continue to be supported.

<p align="center">
    <img src="/img/rtmps/facebook-not-support-rtmp.jpg" />
    <i>Facebook deprecated RTMP, Pages & workplace on Nov, 2019</i>
</p>

Live streaming to facebook need RTMPS, but unfortunately most [FFmpeg](http://ffmpeg.org/) not enable [openssl](http://www.openssl.org/) which need.

### Install the dependencies

```bash
apt-get -y install build-essential autoconf automake cmake libtool git checkinstall nasm yasm libass-dev libfreetype6-dev libsdl2-dev libtool libva-dev libvdpau-dev libvorbis-dev libxcb1-dev libxcb-shm0-dev libxcb-xfixes0-dev pkg-config texinfo wget zlib1g-dev libchromaprint-dev frei0r-plugins-dev ladspa-sdk libcaca-dev libcdio-paranoia-dev libcodec2-dev libfontconfig1-dev libfreetype6-dev libfribidi-dev libgme-dev libgsm1-dev libjack-dev libmodplug-dev libmp3lame-dev libopencore-amrnb-dev libopencore-amrwb-dev libopenjp2-7-dev libopenmpt-dev libopus-dev libpulse-dev librsvg2-dev librubberband-dev librtmp-dev libshine-dev libsmbclient-dev libsnappy-dev libsoxr-dev libspeex-dev libssh-dev libtesseract-dev libtheora-dev libtwolame-dev libv4l-dev libvo-amrwbenc-dev libvpx-dev libwavpack-dev libwebp-dev libx264-dev libx265-dev libxvidcore-dev libxml2-dev libzmq3-dev libzvbi-dev liblilv-dev libmysofa-dev libopenal-dev opencl-dev gnutls-dev libfdk-aac-dev
```

### Prepare source code

```bash
wget https://ffmpeg.org/releases/ffmpeg-4.2.1.tar.gz
tar xzvf ffmpeg-4.2.1.tar.gz
cd ffmpeg-4.2.1
```

### Configuration

To use RTMPS, module `openssl` is require.

__Sample 1__ 

```bash
./configure --disable-shared --enable-static --enable-pthreads --enable-gpl --enable-nonfree --enable-libass --enable-libfdk-aac --enable-libfreetype --enable-libmp3lame --enable-libopus --enable-libvorbis --enable-libvpx --enable-libx264 --enable-filters --enable-openssl --enable-runtime-cpudetect --extra-version=patrickz
```

__Sample 2__

```bash
./configure --disable-shared --enable-static --enable-pthreads --enable-nonfree --enable-version3 --enable-hardcoded-tables --enable-avresample --enable-ffplay --enable-gpl --enable-libmp3lame --enable-libopus --enable-librubberband --enable-libsnappy --enable-libtesseract --enable-libtheora --enable-libvorbis --enable-libvpx --enable-libx264 --enable-libx265 --enable-libxvid --enable-libfontconfig --enable-libfreetype --enable-frei0r --enable-libass --enable-libopencore-amrnb --enable-libopencore-amrwb --enable-libopenjpeg --enable-librtmp --enable-libspeex  --disable-libjack --disable-indev=jack --enable-libsoxr --enable-openssl --enable-runtime-cpudetect --extra-version=patrickz
```

### Build

```bash
make -j8
make install
```

> Please note that `make install` may replace (`ffmpeg`) old one or affect your current version. You can use absolute path (i.e. `<path>/ffmpeg-4.2.1/ffmpeg`) until you sure that it work as expected. 

<p align="center">
    <img src="/img/rtmps/facebook-api-rtmps.png" />
    <i>Facebook Dashboard</i>
</p>


### Test 1

```bash
./ffmpeg -f lavfi -re -i "life=s=300x200:mold=10:r=25:ratio=0.1:death_color=#C83232:life_color=#00ff00,scale=1200:800:flags=16" -f lavfi -re -i sine=frequency=1000:sample_rate=44100 -pix_fmt yuv420p -c:v libx264 -b:v 1000k -g 30 -keyint_min 120 -profile:v baseline -preset veryfast -c:a aac -b:a 96k  -f flv "rtmps://live-api-s.facebook.com:443/rtmp/{YOUR STREAM KEY}"
```

I use [`FFmpeg Filters`](http://ffmpeg.org/ffmpeg-filters.html) to simulate as video source. Don't forget change `{YOUR STREAM KEY}` to your stream key then enter the command.

<p align="center">
    <img src="/img/rtmps/facebook-rtmps-cmd.png" />
    <i>FFmpeg stream to facebook</i>
</p>

If you see screen as above, seem it work and you could see incoming video stream on Facebook Live control panel.

<p align="center">
    <img src="/img/rtmps/facebook-live-panel.png" />
    <i>Facebook live control panel</i>
</p>

### Test 2

```bash
 ffmpeg -re -y -i <input-file.mp4> -ac 1 -ar 44100 -b:a 256k -vcodec libx264 -pix_fmt yuv420p -vf scale=1280:720 -bufsize 1024k -r 30 -g 60 -f flv "rtmps://live-api-s.facebook.com:443/rtmp/{YOUR STREAM KEY}"
```

Now you can live streaming to facebook with RTMPS. Hope you enjoy. Have a nice day!