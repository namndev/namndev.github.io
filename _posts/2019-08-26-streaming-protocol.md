---
layout: post
title: "Streaming Protocols: Everything You Need to Know"
tags: [Video, Live, RTMP, HLS]
comments: true
---

# What Is a Protocol?
A protocol is a [set of rules governing how data travels](https://www.britannica.com/technology/protocol-computer-science) from one communicating system to another. These are layered on top of one another to form a protocol stack. That way, protocols at each layer can focus on a specific function and cooperate with each other. The lowest layer acts as a foundation, and each additional layer adds complexity.

You’ve likely heard of an IP address, which stands for Internet Protocol. This protocol structures how devices using the internet communicate. The Internet Protocol sits at the network layer and is typically overlaid by the Transmission Control Protocol (TCP) at the transport layer, as well as the Hypertext Transfer Protocol (HTTP) at the application layer.

<p align="center">
    <img src="/img/2019/Protocol-Layers_720x594-1.png" />
</p>

The seven layers — which include physical, data link, network, transport, session, presentation, and application — were defined by the [International Organization for Standardization’s (IS0’s) Open Systems Interconnection model](http://www.bitsavers.org/pdf/datapro/communications_standards/2783_ISO_OSI.pdf), as depicted above.

# What Is a Streaming Protocol?
Each time you watch a live stream or video on demand, streaming protocols are used to deliver data over the internet. These can sit in the application, presentation, and session layers.

[Online video delivery uses both streaming protocols and HTTP-based protocols](https://www.streamingmedia.com/Articles/ReadArticle.aspx?ArticleID=84496). Streaming protocols like [Real-Time Messaging Protocol (RTMP)](https://www.wowza.com/blog/rtmp-streaming-real-time-messaging-protocol) enable speedy video delivery using dedicated streaming servers, whereas HTTP-based protocols rely on regular web servers to optimize the viewing experience and quickly scale. Finally, a handful of emerging HTTP-based technologies like the [Common Media Application Format (CMAF)](https://www.wowza.com/blog/what-is-cmaf) and [Apple’s Low-Latency HLS](https://www.wowza.com/blog/apple-low-latency-hls) seek to deliver the best of both options to support low-latency streaming at scale.

# UDP vs. TCP: A Quick Background
User Datagram Protocol (UDP) and Transmission Control Protocol (TCP) are both core components of the internet protocol suite, residing in the transport layer. The protocols used for streaming sit on top of these. UDP and TCP differ in terms of quality and speed, so it’s worth taking a closer look.

The primary difference between UDP and TCP hinges on the fact that TCP requires a three-way handshake when transporting data. The initiator (client) asks the accepter (server) to start a connection, the accepter responds, and the initiator acknowledges the response and maintains a session between either end. For this reason, TCP is quite reliable and can solve for packet loss and ordering. UDP, on the other hand, starts without requiring any handshake. It transports data regardless of any bandwidth constrains, making it speedier and riskier. Because UDP doesn’t support retransmissions, packet ordering, or error-checking, there’s potential for a [network glitch to corrupt the data en route](https://medium.com/@eyevinntechnology/achieving-low-latency-video-streaming-b0b806dcc282).

Protocols like Secure Reliable Transport (SRT) often use UDP, whereas HTTP-based protocols use TCP.


# Considerations When Choosing a Streaming Protocol
Selecting the right protocol starts with defining what you’re trying to achieve. [Latency](https://www.wowza.com/blog/what-is-low-latency-and-who-needs-it), playback compatibility, and viewing experience can all be impacted. What’s more, content distributors don’t always stick with the same protocol from capture to playback. Many broadcasters use RTMP to get from the encoder to server and then [transcode](https://www.wowza.com/blog/what-is-transcoding-and-why-its-critical-for-streaming) the stream into an adaptive HTTP-based format.

# What Are the Most Common Protocols Used for Streaming?
 
__Traditional Streaming Protocols__

- [RTMP (Real-Time Messaging Protocol)](#adobe-rtmp)
- [RTSP (Real-Time Streaming Protocol)/RTP (Real-Time Transport Protocol)](#rtsp/rtp)

__HTTP-Based Adaptive Protocols__

- [Apple HLS (HTTP Live Streaming)](#apple-hls)
- [Low-Latency HLS](#low-latency-hls)
- [MPEG-DASH (Moving Picture Expert Group Dynamic Adaptive Streaming over HTTP)](#mpeg-dash)
- [Low-Latency CMAF for DASH (Common Media Application Format for DASH)](#low-latency-cmaf-for-dash)
- [Microsoft Smooth Streaming](#microsoft-smooth-streaming)
- [Adobe HDS (HTTP Dynamic Streaming)](#adobe-hds)
 
__New Technologies__
- [SRT (Secure Reliable Transport)](#srt)
- [WebRTC (Web Real-Time Communications)](#webrtc)


# Traditional Streaming Protocols
Traditional streaming protocols, such as RTSP and RTMP, support low-latency streaming. But they aren’t supported on all endpoints (e.g., iOS devices). These work best for streaming to a small audience from a dedicated media server.

<p align="center">
    <img src="/img/2019/latency-continuum-with-protocols-1140x638-1.png" />
</p>

As shown above, RTMP delivers video at roughly the same pace as a cable broadcast — in just over five seconds. RTSP/RTP is even quicker at around two seconds. These protocols achieve this by transmitting the data using a firehose approach rather than requiring local download or caching. But because very few players support these protocols, they aren’t optimized for great viewing experiences at scale. Many broadcasters choose to transport live streams to the media server using a stateful protocol like RTMP and then [transcode](https://www.wowza.com/live-video-streaming/complete-guide-to-live-streaming/transcoding) it into an HTTP-based technology for multi-device delivery.

## Adobe RTMP
Adobe designed the [RTMP specification](https://www.wowza.com/blog/rtmp-streaming-real-time-messaging-protocol) for the transmission of audio and video data between technologies like a dedicated streaming server and the Adobe Flash Player. Reliable and low-latency, this worked great for live streaming. But open standards and adaptive bitrate streaming eventually edged RTMP out. The writing on the wall came when [Adobe announced the death of Flash](https://www.forbes.com/sites/tonybradley/2017/07/29/the-death-of-adobe-flash-is-long-overdue/#7239f676f8b3) — slated for 2020.

While Flash’s end-of-life date is overdue, the same cannot be said for using RTMP for video contribution. RTMP encoders are still a go-to for many content producers even though the proprietary protocol has fallen out of favor for last-mile delivery.

<p align="center">
    <img src="/img/2019/Protocols-Workflow-1.png" />
</p>

- [__Audio Codecs__](https://www.wowza.com/blog/best-audio-codecs-live-streaming): AAC, AAC-LC, HE-AAC+ v1 & v2, MP3, Speex, Opus, Vorbis
- [__Video Codecs__](https://www.wowza.com/blog/video-codecs-encoding): H.264, VP8, VP6, Sorenson Spark®, Screen Video v1 & v2
- __Playback Compatibility__: Not widely supported (Flash Player, Adobe AIR, RTMP-compatible players)
- __Benefits__: Low-latency and requires no buffering
- __Drawbacks__: Not optimized for quality of experience or scalability
- __Latency__: 5 seconds
- __Variant Formats__: RTMPT (tunneled through HTTP), RTMPE (encrypted), RTMPTE (tunneled and encrypted), [RTMPS (encrypted over SSL)](https://www.wowza.com/blog/facebook-live-rtmps), [RTMFP (travels over UDP instead of TCP)](https://blog.stackpath.com/rtmp/)

## RTSP/RTP

Like RTMP, RTSP/RTP describes a stateful protocol used for video contribution as opposed to multi-device delivery. While RTMP is a presentation-layer protocol that lets end users command media servers via pause and play capabilities, RTP is a transport protocol used to move said data. It’s supported by UDP at this same layer.

Android and iOS devices don’t have RTSP compatible players out of the box, making this another protocol that’s rarely used for playback.
- [__Audio Codecs__](https://www.wowza.com/blog/best-audio-codecs-live-streaming): AAC, AAC-LC, HE-AAC+ v1 & v2, MP3, Speex, Opus, Vorbis
- [__Video Codecs__](https://www.wowza.com/blog/video-codecs-encoding): H.265 (preview), H.264, VP9, VP8
- __Playback Compatibility__: Not widely supported (Quicktime Player and other RTSP/RTP-compliant players, VideoLAN VLC media player, 3Gpp-compatible mobile devices)
- __Benefits__: Low-latency and requires no buffering
- __Drawbacks__: Not optimized for quality of experience and scalability
- __Latency__: 2 seconds
- __Variant Formats__: The entire stack of RTP, RTCP (Real-Time Control Protocol), and RTSP is often referred to as RTSP
 
# Adaptive HTTP-Based Streaming Protocols
[Streams deployed over HTTP are not technically “streams.”](http://www.onlinevideo.net/2011/05/streaming-vs-progressive-download-vs-adaptive-streaming/) Rather, they’re progressive downloads sent via regular web servers. Using [adaptive bitrate streaming](https://www.wowza.com/blog/adaptive-bitrate-streaming), HTTP-based protocols deliver the best video quality and viewer experience possible — no matter the connection, software, or device. Some of the most common HTTP-based protocols include MPEG-DASH and Apple’s HLS.


## Apple HLS
Since Apple’s a major player in the world of internet-connected devices, it follows that [Apple’s HLS protocol](https://www.wowza.com/blog/hls-streaming-protocol) rules the digital video landscape. For one, the protocol supports [adaptive bitrate streaming](https://www.wowza.com/blog/adaptive-bitrate-streaming), which is key to viewer experience. More importantly, a stream delivered via HLS will play back on the majority of devices — thereby expanding your audience.

While HLS support was initially limited to iOS devices such as iPhones and iPads, native support has since been added to a wide range of platforms. All Google Chrome browsers, as well as Android, Linux, Microsoft, and MacOS devices can play streams delivered using HLS.

- [__Audio Codecs__](https://www.wowza.com/blog/best-audio-codecs-live-streaming): AAC-LC, HE-AAC+ v1 & v2, MP3
- [__Video Codecs__](https://www.wowza.com/blog/video-codecs-encoding): H.265, H.264
- __Playback Compatibility__: Great (All Google Chrome browsers; Android, Linux, Microsoft, and MacOS devices; several set-top boxes, smart TVs, and other players)
- __Benefits__: Adaptive bitrate and widely supported
- __Drawbacks__: Quality of experience is prioritized over low latency
- __Latency__ : 6-30 seconds (lower latency only possible when tuned)
- __Variant Formats__: Low-Latency HLS (see below), PHLS (Protected HTTP Live Streaming)

## Low-Latency HLS
Mid 2019, [Apple announced an extension to their HLS protocol](https://www.wowza.com/blog/apple-low-latency-hls) designed to drive latency down at scale. The protocol achieves this using HTTP/2 PUSH delivery combined with shorter media chunks. Unlike standard HLS, Apple Low-Latency HLS doesn’t yet support adaptive bitrate streaming — but it is on the roadmap.

- __Playback Compatibility__: Any players that aren’t optimized for Low-Latency HLS can fall back to standard (higher-latency) HLS behavior
- __Benefits__: Low latency meets HTTP-based streaming
- __Drawbacks__: As an emerging spec, vendors are still implementing support
- __Latency__: 3 seconds or less

> Note: Since posting this blog, the [Low-Latency HLS](https://www.wowza.com/blog/apple-low-latency-hls) specification has been incorporated into the [overarching HLS standard](https://www.wowza.com/blog/hls-streaming-protocol). To learn more, [check out our article on the update](https://www.wowza.com/blog/ietf-incorporates-low-latency-hls-into-the-hls-spec).

## MPEG-DASH
When it comes to MPEG-DASH, the acronym spells out the story. The Moving Pictures Expert Group (MPEG), an international authority on digital audio and video standards, developed Dynamic Adaptive Streaming over HTTP (DASH) as an industry-standard alternative to HLS. Basically, with DASH you get an open-source option. But because Apple tends to priorities its proprietary software, support for DASH plays second fiddle.
- [__Audio Codecs__](https://www.wowza.com/blog/best-audio-codecs-live-streaming): Codec-agnostic
- [__Video Codecs__](https://www.wowza.com/blog/video-codecs-encoding): Codec-agnostic
- __Playback Compatibility__: Good (All Android devices; most post-2012 Samsung, Philips, Panasonic, and Sony TVs; Chrome, Safari, and Firefox browsers)
- __Benefits__: Vendor independent, international standard for adaptive bitrate
- __Drawbacks__: Not supported by iOS or Apple TV
- __Latency__: 6-30 seconds ([lower latency only possible when tuned](https://www.wowza.com/docs/how-to-configure-mpeg-dash-packetization-mpegdashstreaming))
- __Variant Formats__: MPEG-DASH CENC (Common Encryption)

## Low-Latency CMAF for DASH
The Common Media Application Format, or [CMAF](https://www.wowza.com/blog/what-is-cmaf), is in itself is a media format. But when paired with [chunked encoding](https://www.wowza.com/blog/low-latency-cmaf-chunked-transfer-encoding#chunked-encoding) and [chunked transfer encoding](https://www.wowza.com/blog/low-latency-cmaf-chunked-transfer-encoding#chunked-transfer-encoding) for delivery over DASH, it should support sub-three-second delivery. While its transfer setup differs from that of Low-Latency HLS, the use of shorter data segments is quite similar.

- __Playback Compatibility__: Any players that aren’t optimized for low-latency CMAF for DASH can fall back to standard (higher-latency) DASH behavior
- __Benefits__: Low latency meets HTTP-based streaming
- __Drawbacks__: As an emerging spec, vendors are still implementing support
- __Latency__: 3 seconds or less

## Microsoft Smooth Streaming
Microsoft developed Microsoft Smooth Streaming for use with Silverlight player applications. It enables adaptive delivery to all Microsoft devices.
- [__Audio Codecs__](https://www.wowza.com/blog/best-audio-codecs-live-streaming): AAC, MP3, WMA
- [__Video Codecs__](https://www.wowza.com/blog/video-codecs-encoding): H.264, VC-1
- __Playback Compatibility__: Good (Microsoft and iOS devices, Xbox, many smart TVs)
- __Benefits__: Adaptive bitrate and supported by iOS
- __Drawbacks__: Proprietary technology
- __Latency__: 6-30 seconds ([lower latency only possible when tuned](https://www.wowza.com/docs/how-to-configure-microsoft-smooth-streaming-packetization-smoothstreaming))
 
## Adobe HDS
HDS was developed for use with Flash Player applications as the first adaptive bitrate protocol. Because the end-of-life date for Flash is looming, it’s fallen out of favor.
- [__Audio Codecs__](https://www.wowza.com/blog/best-audio-codecs-live-streaming): AAC, MP3
- [__Video Codecs__](https://www.wowza.com/blog/video-codecs-encoding): H.264, VP6
- __Playback Compatibility__: Not widely supported (Flash Player, Adobe AIR)
- __Benefits__: Adaptive bitrate technology for Flash
- __Drawbacks__: Proprietary technology with lacking support
- __Latency__: 6-30 seconds ([lower latency only possible when tuned](https://www.wowza.com/docs/how-to-configure-adobe-hds-packetization-sanjosestreaming))

# Emerging Technologies
Last but not least, new technologies like WOWZ, WebRTC, and SRT were designed with latency in mind. Similar to low-latency CMAF for DASH and Apple Low-Latency HLS, these protocols continue to evolve.
 
## SRT
This open-source protocol is recognized as a proven alternative to proprietary transport technologies — helping to deliver reliable streams, regardless of network quality. From recovering lost packets to preserving timing behavior, [SRT](https://www.wowza.com/low-latency/SRT-secure-reliable-transport) was designed to solve the challenges of video contribution and distribution across the public internet. Although it’s quickly taking the industry by storm, SRT is most often used for first-mile contribution rather than last-mile delivery.

- [__Audio Codecs__](https://www.wowza.com/blog/best-audio-codecs-live-streaming): Codec-agnostic
- [__Video Codecs__](https://www.wowza.com/blog/video-codecs-encoding): Codec-agnostic
- __Playback Compatibility__: Limited (currently used for contribution)
- __Benefits__: High-quality, low-latency video over suboptimal networks
- __Drawbacks__: Playback support is still in the works
- __Latency__: 3 seconds or less


## WebRTC
[WebRTC](https://www.wowza.com/blog/what-is-webrtc) is a combination of standards, protocols, and JavaScript APIs that enables real-time communications (RTC, hence its name). Users connecting via Chrome, Firefox, or Safari can communicate directly through their browsers — enabling sub-500 millisecond latency. [According to Google](https://blog.chromium.org/2020/05/celebrating-10-years-of-webm-and-webrtc.html), “with Chrome, Edge, Firefox, and Safari supporting WebRTC, more than 85% of all installed browsers globally have become a client for real-time communications on the internet.”
- [__Audio Codecs__](https://www.wowza.com/blog/best-audio-codecs-live-streaming): Opus, iSAC, iLBC
- [__Video Codecs__](https://www.wowza.com/blog/video-codecs-encoding): H.264, VP8, VP9
- __Playback Compatibility__: Chrome, Firefox, and Safari support WebRTC without any plugin
- __Benefits__: Super fast and browser-based
- __Drawbacks__: Designed for video conferencing and not scale
- __Latency__: Sub-500-millisecond delivery


# What Protocols Are Not: Codecs
For anyone who wasn’t sure what I was talking about when listing the audio and video codecs for each protocol, this video will walk you through the difference.

# Conclusion: How to Choose the Right Protocol
Protocols differ in the following areas:
- Scalability
- Latency
- Quality of experience (adaptive bitrate enabled, etc.)
- Use (first-mile contribution vs. last-mile delivery)
- Playback support
- Proprietary vs. open source
- Codec requirements

By prioritizing the above considerations, it’s easy to narrow down what’s best for you.

RTMP and SRT are great bets for first-mile contribution, while both DASH and HLS lead the way when it comes to playback. That’s why we’re especially excited to see low-latency CMAF for DASH and Low-Latency HLS take off. But you may be looking to deploy a one-to-few conference, in which case WebRTC would be better suited.

[Read more..](https://www.wowza.com/blog/streaming-protocols)