---
layout: post
title: "Mastering HTTP Live Streaming"
tags: [Video, Live, Codec, HLS]
comments: true
---

HTTP Live Streaming (HLS) sends audio and video over HTTP from an ordinary web server for playback on iOS-based devices. HLS is designed for reliability and dynamically adapts to network conditions by optimising playback for the available speed of wired and wireless connections.

>Note: If your app delivers video content longer then 10 minutes or greater than 5MB of data, the App Store requires you to use HLS.

If your app uses HLS over cellular networks, you must provide at least one stream at 192Kbps or lower bandwidth. The low-bandwidth stream may be audio-only or audio with a still image.

# Why HTTP Live Streaming?

In earlier days, streaming a video over a network was not an easy job to do. If you have a single HD file on your server and you have to play that video in a mobile that only has the 2G network bandwidth, just think how much time you have to wait to see the entire video. Also, we need a streaming server to serve the video to users.

HLS help us easily deploy media content in streams using common place web servers rather than specialized streaming servers. Lets discuss the key features of HLS

- Adaptive streaming
- Content protection
- Closed captions and subtitles
- Ad Insertion
- Fast forward and reverse playback
- Alternate audio
- Fallback with Stream Alternates

## Adaptive streaming

The basic idea behind adaptive streaming is to generate various versions of the media file on different resolutions and bitrate (bps in a video/audio). Then choose one of them based on the user’s bandwidth, screen size, and other factors.

Those different versions of the video files are split into multiple chunks that are then played according to the user’s bandwidth. You can clearly see how the adaptive streaming in HLS works from the image.

<p align="center">
    <img src="/img/2019/http-live-streaming-1.png" />
    <p align="center">Adaptive streaming in HLS</p>
</p>

Based on a wide range of user’s bandwidth environment (2G, 3G, 4G, LTE, WIFI Low, WIFI High), the video can be played with respective HLS bitrate versions 240p, 360p, 480p, 720p, and 1080p. This also allows the user to switch from one bitrate version to another, which will give the seamless streaming experience to the user.

## Content protection

Media segments can be individually encrypted using sample-level encryption. References to the corresponding key files appear in the playlist file so the player can retrieve the keys for decryption.

HLS supports key exchange with the method of your choice. Static keys, encoder generated keys, and frequently updated keys are just a few of the possibilities.

# What Does HLS Support?
HLS supports the following:

- Live broadcasts
- Prerecorded content: video on demand (VOD)
- Alternate streams: multiple alternate streams at different bit rates
- Intelligent switching: intelligent switching of streams in response to network bandwidth changes
- Media encryption: encrypt the video content
- User authentication: deliver the video/audio content with user authentication

# Does HLS use TCP or UDP as its transport protocol?

TCP and UDP are transport protocols, meaning they are responsible for delivering content over the Internet. TCP tends to deliver data more reliably than UDP, but the latter is much faster, even though some data may be lost in transit.

Because UDP is faster, many streaming protocols use UDP instead of TCP. HLS, however, uses TCP. This is for several reasons:

1. HLS is over HTTP, and the HTTP protocol is built for use with TCP (with some exceptions).
2. The modern Internet is more reliable and more efficient than it was when streaming was first developed. In many parts of the world today, user connectivity has vastly improved, especially for mobile connections. As a result, users have enough bandwidth to support the delivery of every video frame.
3. Adaptive bitrate streaming helps compensate for the potentially slower data delivery of TCP.
4. HLS streaming does not need to be "real time," as is the case with videoconferencing connections. A few extra seconds of lag does not impact the user experience as much as missing video frames would.

<p align="center">
    <img src="/img/2019/http-live-streaming-2.png" />
    <p align="center">Components of HTTP Live Streaming</p>
</p>

# Server Component
The server component consists of three important things, using these we will create the master playlist file (.m3u8)

- Encoder : Converting data from one format to another using defined compression mechanism
- Transcoder: Creating multiple quality variants of the same content.
- Stream Segmenter: Breaking the content into a series of short media files. The video is divided up into segments a few seconds in length. The length of the segments can vary, although the default length is 10 seconds.
    - In addition to dividing the video into segments, HLS creates an index file of the video segments to record the order they belong in.
    - HLS will also create several duplicate sets of segments at different quality levels: 480p, 720p, 1080p, and so on.


# Distribution Component
The distribution system is generally a combination web server and a web-caching system (CDN) that delivers the media files and index files to the client over HTTP.

- Origin web server : System that delivers the media files and index files of the client over HTTP
- CDN (Content delivery network)

# Client (Player)
The player is a client here, it’s responsible for playing the HLS content in the user’s device. The player begins by fetching the content of the .m3u8 file, using the media playlist URL’s it will download the media chunks in sequence and reassembles them to present the continuous streaming to the user.

This process continues until the player encounters an end tag in the .m3u8 file. If no end tag is present, the .m3u8 file is part of an ongoing broadcast i.e live event.

During ongoing broadcasts, the player loads a new version of the .m3u8 file periodically to download updated media chunks.

<p align="center">
    <img src="/img/2019/http-live-streaming-3.png" />
    <p align="center">AVPlayer for play the HLS video</p>
</p>

# M3U8 Playlist
This image shows how the index file (master.m3u8) was constructed and these are the important tags we need to know about the HSL.

<p align="center">
    <img src="/img/2019/http-live-streaming-4.png" />
    <p align="center">.m3u8 file structure</p>
</p>

__EXTM3U:__ Indicates that the playlist is an extended M3U file. All HLS playlists must start with this tag.

__EXT-X-PLAYLIST-TYPE:__ This tag may contain a value of either EVENT (Live) or VOD. If the tag is present and has a value of EVENT, the server must not change or delete any part of the playlist file. If the tag is present and has a value of VOD, the playlist file must not change.

__EXT-X-TARGETDURATION:__ Specifies the maximum media-file duration.

__EXT-X-VERSION:__ HLS protocol version.

__EXT-X-MEDIA-SEQUENCE:__ Indicates the sequence number of the first URL that appears in a playlist file.

__EXTINF:__ A record marker that describes the media file identified by the URL that follows it. Each media file URL must be preceded by an EXTINF tag.

__EXT-X-STREAM-INF:__ Indicates that the next URL in the playlist file identifies another playlist file, parameter defines stream’s metadata like BANDWIDTH, AVERAGE-BANDWIDTH, RESOLUTION, CODECS, etc.

__EXT-X-ENDLIST:__ Indicates that no more media files will be added to the playlist file.

The next step will be how to integrate HLS with your app. I think this deserves a separate post. I’m planning to write one soon, keep watching this space for updates.

[Read more...](https://medium.com/better-programming/mastering-http-live-streaming-d540caa4a9f4)