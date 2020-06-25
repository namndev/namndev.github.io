
---
layout: post
title: "Best Audio Codecs for Live Video Streaming"
tags: [Video, Live, Codec, HLS]
comments: true
---

# Encoding and Data Compression
Ever wonder why vinyl albums sound better than MP3 recordings? Or is that just something pretentious hipsters say to channel John Cusack’s character in High Fidelity?

As it turns out, this one isn’t up for debate. The sound quality of records far outweighs that of most digital formats today. And we’re not just talking about the crackling noise a needle makes while gliding across the record’s surface.

The difference comes down to file size. LPs include more audio data than MP3 files of the same track, which is why they also take up more space.

MP3 files, on the other hand, are compressed via a process called [encoding](https://www.wowza.com/blog/video-codecs-encoding). A compression tool called a codec is used to discard all audio components beyond the limitations of human hearing. This allows the encoded audio files to be more easily transported across the internet or stored on a mobile device.

Encoding can occur on a digital camera, via a stand-alone appliance, as part of a computer software, or in a mobile app. And it’s essential to [streaming](https://www.wowza.com/blog/live-video-streaming-how-it-works). Without encoding, bandwidth limitations and bulky file sizes would get in the way.
 
# Lossy and Lossless Compression
__Lossless compression__ occurs when the data is compacted without any information being discarded. A trusty ZIP file does just this, allowing us to cram a bunch of information into a small amount of space while preserving data integrity.

Audio codecs that use lossless compression include WMA Lossless (Windows Media Audio), ALAC (Apple Lossless Audio Codec), and FLAC (Free Lossless Audio Codec).

This works great for storing crisp audio on a Blu-ray disk or CD. Once delivered and decoded, the same exact audio file plays back. But the big files created via lossless compression are difficult to transport for live video streaming.

__Lossy compression__, on the other hand, enables a much greater reduction in file size by tossing out unnecessary data. Size is important when it comes to streaming, making lossy compression a must.

__The goals of lossy compression are threefold:__

1. Drop all data that would go undetected by the human ear
2. Reduce the quality of hard-to-hear sounds
3. Quickly compress all remaining data

Literally ‘coder-decoder’ or ‘compressor-decompressor,’ codecs apply algorithms to the video and create a facsimile of it. The video is shrunk down for storage and transmission, and later decompressed for viewing.

<p align="center">
    <img src="/img/2019/Encoding.gif" />
</p>

# Audio Codecs for Live Streaming
Trying to pick the best audio codec for your live streams? Here’s our list of the top choices based on quality and compatibility:

1. [AAC](#aac)
2. [MP3](#mp3)
3. [Opus](#opus)
4. [Vorbis](#vorbis)
5. [Speex](#speex)
6. [G.711](#g711)
7. [AC3](#ac3)

## __1. AAC: Advanced Audio Coding__

Defined by MPEG-4, AAC is the most common audio codec. It improved upon the MP3 codec by offering the same audio quality at a lower bitrate. As long as you aren’t looking for an exact replication, AAC works great. This widely supported standard is used by YouTube, Android, iOS, and iTunes.

Two extensions of AAC exist: HE-ACC for low bitrates and AAC-LC for low delay. We suggest using HE-AAC whenever bandwidth is a concern, and AAC-LC for two-way communication.

## __2. MP3: MPEF-1 Audio Layer III__

The MP3 codec revolutionized music consumption in the 1990s, prompting millennials near and far to swap their CD players for iPods. Nearly every audio-supported digital device in the world can play back the MP3 format, making it a viable option for live streaming. But because AAC offers superior compression, we’d recommend going with that.

## __3. Opus__

Developed by the Xiph.Org foundation, Opus provides [higher quality audio than any other lossy audio format](http://opus-codec.org/comparison/). It’s open-source and royalty-free, but has yet to be widely adopted.

## __4. Vorbis__

Also a non-proprietary alternative, Vorbis was designed to compete against closed codecs like AAC. Again, this codec lacks native support across devices. And because Opus outperforms Vorbis as far as quality goes, there’s no benefit to taking this route.

## __5. Speex__

The Xiph.Org foundation designed this patent-free codec as an alternative to proprietary speech codecs. Like Vorbis, though, the Speex codec has been made obsolete by Opus.

## __6. G.711__

G.711 was first released back in 1972. The codec works fine for real-time communication, but it isn’t exactly broadcast-quality.
G.711 is often used for VoIP calls made to and from regular phones.

## __7. AC-3 and E-AC-3: Dolby Digital and Dolby Digital Plus__

The AC-3 and E-AC-3 formats are less advanced than AAC. The only benefit of these audio codecs is backwards compatibility on Dolby Digital equipment.
 

# Multi-Codec Video Delivery
For compatibility and quality, your best bet is AAC. But because proprietary codecs and video containers exist, it can be helpful to deliver multiple different versions of your live streams to viewers.

We offer the [Wowza Streaming Engine™ software](https://www.wowza.com/products/streaming-engine) for multi-codec video delivery using your own servers — whether they’re on premises or in a third-party cloud platform. The [Wowza Streaming Cloud™ service](https://www.wowza.com/products/streaming-cloud) can also intake several different formats and transcode MP3s into the AAC format.
 
__Wowza Streaming Engine Supported Formats__

__Input__: AAC, AAC-LC, HE-AAC (ACC+ or accPlus), HE-AACv2 (enhanced AAC+, aacPlus v2), MP3, Speex, Opus, Vorbis, AC-3, E-AC-3.

__Output__:  AAC, AAC-LC, HE-AAC (ACC+ or accPlus), HE-AACv2 (enhanced AAC+, aacPlus v2), Opus, G.711
 
__Wowza Streaming Cloud Supported Formats__

__Input__: AAC, AAC-LC, HE-AAC (ACC+ or accPlus), HE-AACv2 (enhanced AAC+, aacPlus v2), MP3

__Output__: AAC