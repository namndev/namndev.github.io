---
layout: post
title: "Video Codecs and Encoding: Everything You Should Know"
tags: [Video, Live, Codec, HLS]
comments: true
---


Without video codecs, “Netflix and chill” wouldn’t have ever been coined. These two-part compression tools allow distributors to condense a video file for delivery across the internet via a process called video encoding. Codecs are the reason we can so easily stream videos and FaceTime loved ones, even with limited bandwidth.

Thanks to codecs, Netflix manages to stream more than 97,000 hours of content every minute.[¹](https://web-assets.domo.com/blog/wp-content/uploads/2018/06/18_domo_data-never-sleeps-6verticals.pdf)  And in order to get those streams into your living room — no matter the device — Netflix must [use both new and tried-and-true codecs](http://www.dtcreports.com/weeklyriff/2018/04/26/netflix-well-leave-no-codec-or-customer-behind/).

What does video encoding involve and how do video codecs work? We’ll dig deeper below and cover the [best video codecs for streaming](https://www.wowza.com/blog/video-codecs-encoding#best-codecs).

# What Is Video Encoding?
Video encoding refers to the process of converting raw video into a digital format that’s compatible with many devices. When it comes to streaming, videos are often compressed from gigabytes of data down to megabytes of data. Encoding can occur on a digital camera, via a stand-alone appliance, as part of a computer software, or in a mobile app. Codecs are used to digitally compress the video.

We offer the Wowza Clearcaster™ live encoding appliance which enables production-quality encoding with unmatched cloud control.
 
# What Is a Codec?
To shrink a video into a more manageable size, content distributors use a video compression technology called a codec. Codecs allow us to tightly compress a bulky video for delivery and storage.

Literally ‘coder-decoder’ or ‘compressor-decompressor,’ codecs apply algorithms to the video and create a facsimile of it. The video is shrunk down for storage and transmission, and later decompressed for viewing.

Streaming employs both audio and video codecs. H.264, also known as AVC, is the most common video codec. AAC is the most common audio codec.

<p align="center">
    <img src="/img/2019/Encoding.gif" />
</p>

# What Is a Video Container Format?
Once compressed, the components of a stream are packaged into a wrapper or file format. These files contain the audio codec, video codec, closed captioning, and any associated metadata. Common containers include .mp4, .mov, .ts, and .wmv.

Containers can often input multiple types of codecs. That said, not all playback platforms accept all containers and codecs. That’s why multi-format encoding is crucial when streaming to a wide range of devices.

For example: a .mov file and a .wmv file might have the same exact data and codecs inside. But the .mov file would be used for playback on Macbook’s QuickTime player, while the .wmv file would be used for playback on a PC’s Windows Media Player.
 
# Difference Between Video Codecs and Containers
A codec acts upon the video, both at the source to compress it and before playback to decompress it. This is done through lossy compression, during which any unnecessary data is discarded.

Lossy compression is a lot like Wonkavision in Charlie and the Chocolate Factory. It makes a large collection of data smaller for transport to your screen:

A __video container format__, on the other hand, stores the video codec, audio codec, and metadata such as subtitles or preview images. The container holds all the components together and determines which programs can accept the stream.
 
# Best Video Codecs for Streaming
Streaming will soon replace traditional broadcast. And delivering video over the internet to a variety of devices starts with encoding with a variety of codecs. Next-generation codecs improve encoding efficiency and quality, while legacy codecs enable playback on outdated devices.

Take it from the largest distributor of video streaming around: Netflix.

> “Netflix says it utilizes a deep toolbox of codecs, which can be called upon to stream compatible formats to display devices. Although Netflix continually adds new and improved codecs, it has never abandoned one — it continues to support the VC1 codec it started with in the first Netflix streaming device, a 10-year-old LG Blu-ray player.”[²](http://www.dtcreports.com/weeklyriff/2018/04/26/netflix-well-leave-no-codec-or-customer-behind/)

Our list of the best video codecs available today includes both the old and the new. While industry leaders continue to refine and develop the latest compression tools, they also employ older codecs like H.264/AVC for delivery to legacy devices.

__Let’s look closer at the most common encoding technologies in 2019.__

1. [H.264/AVC](#h264avc)
2. [H.265/HEVC](#h265hevc)
3. [AV1](#av1)
4. [VP9](#vp9)
5. [H.266/VVC](#h266vvc)


## H.264/AVC
The majority of encoding output today takes the form of H.264 files, also referred to as AVC (Advanced Video Coding). This widely supported codec was developed by the International Telecommunications Union and the International Organization for Standardization/International Electrotechnical Commission (ISO/IEC) Moving Picture Experts Group — _wow, what a mouthful_.

H.264 also has significant penetration into markets outside of streaming, such as Blu-ray disks and cable broadcasting. It is often incorporated with the AAC audio codec and can be packaged into .mp4, .mov, .F4v, .3GP, and .ts containers.

H.264 plays on virtually any device, delivers quality video streams, and comes with the least concerns surrounding royalties.

<p align="center">
    <img src="/img/2019/H.264-Compression-1.png" />
</p>

## H.265/HEVC

The ISO/IEV Moving Picture Experts Group developed [H.265](https://www.streamingmedia.com/Articles/Editorial/What-Is-.../What-Is-HEVC-(H.265)-87765.aspx) as the successor to H.264. Also called HEVC (High Efficiency Video Coding), this codec aims to improve compression efficiency and support 8K resolution. It generates smaller files than H.264, thus decreasing the bandwidth required to view these streams. This makes it an ideal codec for high-resolution streaming.

That said, only about 10 percent of encoded files take the form of H.265. [Uncertainties surrounding royalties](https://www.ibc.org/delivery/codec-wars-the-battle-between-hevc-and-av1/2710.article) have stifled adoption. Specifically, content distributors are frustrated by the lacking transparency into what they’ll have to pay when using this codec.

## AV1

Frustrated about the royalties associated with H.265, Amazon, Netflix, Google, Microsoft, Cisco, and Mozilla formed the Alliance for Open Media. The goal? Create an open-source, royalty-free alternative called [AV1](https://www.streamingmedia.com/Articles/Editorial/What-Is-.../What-is-AV1-111497.aspx).

While the codec has been finalized, this initiative to democratize high-quality video delivery and playback is still playing out.

<p align="center">
    <img src="/img/2019/AV1-1.png" />
</p>

According to Johnathan Rosenberg, CTO of the Collaboration Technology Group at Cisco, “The creation of an advanced, royalty-free video codec is paramount to the ongoing success of collaboration products and services. This is why Cisco joined [AOMedia](https://aomedia.org/about/) as a founding member, and why Cisco has invested in making AV1 both efficient and accessible to the internet community.”

AV1 touts itself as being 30 percent more efficient than HEVC, but these claims still need to be verified by independent sources. It will also take some time before AV1 hardware decoding capabilities are integrated on a mass scale. Even Apple devices are lacking support for the codec, despite the fact that Apple joined the Alliance back in January of 2018.

In other words, the industry is still in flux when it comes to AV1.

“The one disadvantage at this point is that [AV1] is just new,” [explains Anne Aaron](http://ibc.gallery.video/ibctv/detail/video/6086151976001/ibc2019-conference:-realising-the-future-of-av1:-video-and-visual-cloud?autoStart=false&q=AV1), director of encoding technologies at Netflix. “H.264 is a really good codec that’s been developed over more than ten years — and AV1 is new, so there’s still kinks in implementations.”

It will take some time before AV1 hardware decoding capabilities are integrated on a mass scale. While leaders at Netflix, Facebook, Google, and more are planning to make the move to AV1, playback limitations can’t be ignored.

## VP9
Google developed [VP9](https://www.streamingmedia.com/Articles/Editorial/What-Is-.../What-Is-VP9-111334.aspx) as a royalty-free alternative to HEVC. Every Android phone and Chrome browser supports VP9, as well as Google’s video platform YouTube. This codec often makes an appearance in [WebRTC](https://www.wowza.com/blog/what-is-webrtc) workflows, with [more than 90% of Chrome-encoded WebRTC video using VP9 or its predecessor VP8](https://blog.chromium.org/2020/05/celebrating-10-years-of-webm-and-webrtc.html)

VP9 offers better quality at the same bitrate as H.265/HEVC. While the majority of browsers support it, Apple devices do not.

VP9 is thought of as AV0, or an earlier version of AV1. And for the time being, it’s also a better alternative to AV1 since more devices support it.

## H.266/VVC
Intended to usurp H.265, VVC (versatile video coding) comes with the same royalty issues as its predecessor.

According to Beamr’s chief technology officer Dror Gill, “It’s okay to pay royalties as long as you know how much you need to pay and when. With H.264, it was very clear how much you need to pay, there was one body collecting all the royalties, and this became the world’s most prominent video codec. The same can happen with VVC, if they get their act together before releasing the standard”[³](http://www.streamingmediaglobal.com/Articles/Editorial/Featured-Articles/Encoding--Transcoding-2018-Part-1-128430.aspx)

Still a nascent technology, we’re waiting to see how adoption of VVC pans out. [This codec isn’t intended to ship until the end of 2020](https://www.streamingmedia.com/Articles/Editorial/Featured-Articles/HEVC-AV1-VVC-How-to-Make-Sense-of-2019s-World-of-Codecs-133589.aspx?pageNum=2) and unforeseen challenges on the licensing front are up in the air.

# Multi-Codec Video Delivery
There you have it. Video codecs are what allow us to take the boundless world, capture a slice of it through the lens of our camera, and compress it down for delivery over the internet.

Because proprietary codecs and video containers exist, it’s important to deliver multiple different versions of your live streams to viewers.

Luckily, we offer the [Wowza Streaming Engine™ software](https://www.wowza.com/products/streaming-engine) for multi-codec video delivery using your own servers — whether they’re on premises or in a third-party cloud platform. For those who want to get up and running quickly without any hassles, the [Wowza Streaming Cloud™ service](https://www.wowza.com/products/streaming-cloud) might be a better fit.

That way, you can convert streams as needed by [transcoding](https://www.wowza.com/live-video-streaming/live-transcoding) the data into new codecs or [transmuxing](https://www.wowza.com/blog/what-the-mux-instead-of-a-flash-alternative-transmux) into different formats.

# Wowza Transcoder Specifications

| # | __Decoding (Inputs)__  |   __Encoding (Outputs)__	|
|:------- |:------:| -----:|
| __Video__ | H.265/HEVC, H.264/AVC, MPEG4 Part 2, MPEG2, VP9, VP8 | H.265/HEVC, H.264/AVC, H.263 (v2), VP9 |
| __Audio__ | 	MP3, AAC, AAC-LC, HE-AAC+ v1 & v2, MPEG1 Part 1/2, Speex, G.711, Opus, Vorbis | AAC, AAC-LC, HE-AAC+ v1 & v2, Opus, G.711 |

__Get your content where it needs to go. Support multi-codec video delivery with Wowza.__