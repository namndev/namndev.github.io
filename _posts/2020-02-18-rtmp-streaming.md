---
layout: post
title: "RTMP Streaming: The Real-Time Messaging Protocol Explained"
tags: [Video, Live, RTMP, HLS]
comments: true
---

In the early days of streaming, the Real-Time Messaging [Protocol](https://www.wowza.com/blog/streaming-protocols) (RTMP) was the de facto standard for transporting video over the internet (or in laymen’s terms, for streaming). RTMP is a TCP-based protocol designed to maintain persistent, low-latency connections — and by extension, smooth streaming experiences.

The protocol started out as the secret sauce behind live and on-demand streaming with [Adobe Flash Player](https://www.adobe.com/products/flashplayer.html). Because this popular [Flash plugin powered 98% of internet browsers in its heyday](https://www.youtube.com/watch?v=5Rv50RCwqo8), RTMP was used ubiquitously.

The majority of encoders today can transmit RTMP, and most media servers can receive it. Even big social media players like [Facebook](https://www.facebook.com/), [YouTube](https://www.youtube.com/), [Twitch](https://www.twitch.tv/), and [Periscope](https://www.pscp.tv/) accept it. However, RTMP streams run into compatibility issues when it comes to playback on popular browsers and devices.

In this article, we’ll take a look at the RTMP specification, the history behind RTMP streaming, and alternative protocols to use.

# What Is RTMP?
The [RTMP specification](https://www.adobe.com/devnet/rtmp.html) is a streaming protocol initially designed for the transmission of audio, video, and other data between a dedicated streaming server and the Adobe Flash Player. While once proprietary, RTMP is now an open specification.

[According to Adobe](https://wwwimages2.adobe.com/content/dam/acom/en/devnet/rtmp/pdf/rtmp_specification_1.0.pdf):
> "Adobe’s Real Time Messaging Protocol (RTMP) provides a bidirectional message multiplex service over a reliable stream transport, such as TCP [RFC0793], intended to carry parallel streams of video, audio, and data messages, with associated timing information, between a pair of communicating peers."


# RTMP Streaming: A Snapshot 
- [__Audio Codecs__](/2019-03-13-best-audio-codec-for-live): AAC, AAC-LC, HE-AAC+ v1 & v2, MP3, Speex, Opus, Vorbis
- [__Video Codecs__](/2019-10-16-video-codecs-encoding): H.264, VP8, VP6, Sorenson Spark®, Screen Video v1 & v2
- __Playback Compatibility__: Not widely supported
Limited to Flash Player, Adobe AIR, RTMP-compatible players
No longer accepted by iOS, Android, most browsers, and most embeddable players
- __Benefits__: Low-latency and minimal buffering
- __Drawbacks__: Not optimized for quality of experience or scalability
- __Latency__: 5 seconds
- __Variant Formats__: RTMPT (tunneled through HTTP), RTMPE (encrypted), RTMPTE (tunneled and encrypted), [RTMPS (encrypted over SSL)](https://www.wowza.com/blog/facebook-live-rtmps), RTMFP (layered over UDP instead of TCP)

# How Does RTMP Streaming Work?
Macromedia (which is today Adobe Systems) developed the [RTMP specification](https://www.adobe.com/devnet/rtmp.html) for high-performance transmission of audio and video data. RTMP maintains a constant connection between the player client and server, allowing the protocol to act as a pipe and rapidly move video data through to the viewer.

Because RTMP sits on top of the Transmission Control Protocol (TCP), it uses a three-way handshake when transporting data. The initiator (client) asks the accepter (server) to start a connection; the accepter responds; then the initiator acknowledges the response and maintains a session between either end. For this reason, RTMP is quite reliable.

History of RTMP Streaming
Flash Player and RTMP were the dominant delivery mechanisms for live streaming up until the early 2010s. When used together, these technologies support lightning-fast video delivery with around five seconds of latency. But HTML5 video streaming, open standards, and adaptive bitrate delivery eventually edged RTMP streaming out when it came to last-mile delivery.

<p align="center">
    <img src="/img/2019/latency-continuum-with-protocols-1140x638-1.png" />
</p>

Why? While RTMP works well, it has historically encountered issues getting past firewalls. And as a stateful protocol, RTMP requires a dedicated streaming server. The writing on the wall came when [Adobe announced the death of Flash](https://www.forbes.com/sites/tonybradley/2017/07/29/the-death-of-adobe-flash-is-long-overdue/#7239f676f8b3) — slated for this year.

The industry now favors HTTP-based (Hypertext Transfer Protocol) protocols that use plain-old web servers. This improves scalability and viewing experience by allowing local servers to cache streaming content. With this move, [adaptive bitrate streaming](https://www.wowza.com/blog/adaptive-bitrate-streaming) has become more common, allowing broadcasters to optimize content for viewers’ devices and connectivity.

[Adobe released RTMP to the public back in 2012](https://users/traci.ruether/Documents/Current%20Projects/Blogs/RTMP/adaptive%20bitrate%20streaming), and 2020 marks the final year that Adobe will update and distribute Flash Player. As such, fewer devices support Flash (and by extension RTMP) than ever before. [In Adobe’s own words](https://theblog.adobe.com/adobe-flash-update/), content distributors are encouraged “to migrate any existing Flash content to… new open formats.”
 
# So, Is RTMP Dead?
Flash’s end-of-life date is overdue. But the same cannot be said for RTMP. RTMP encoders are still a go-to for many content producers, and 33% of respondents to our Video Streaming Latency Report indicated using it.

__Which streaming formats are you currently using?__

<p align="center">
    <img src="/img/2019/Streaming-Formats-Currently-Used_Q8-1-1.png" />
</p>

Many broadcasters choose to transport live streams to their media server using RTMP and then [transcode](https://www.wowza.com/blog/what-is-transcoding-and-why-its-critical-for-streaming) them for delivery to a range of players and devices. In other words, RTMP streaming is alive and well for content contribution — just not last-mile delivery.
 
# Typical RTMP Live Stream Workflow
Content distributors aren’t limited to one streaming protocol from capture to playback. In fact, repackaging a live stream into as many protocols as possible helps ensure broad distribution.   

Because today’s HTML5 players require HTTP-based protocols like [HLS](https://www.wowza.com/blog/hls-streaming-protocol), a [media server](https://www.wowza.com/products/streaming-engine) or [streaming service](https://www.wowza.com/products/streaming-cloud) can be used to ingest an RTMP stream and transcode it into a more playback-friendly alternative. Whether the RTMP stream is coming from an IP camera, mobile app, or [broadcast-grade encoder](https://www.wowza.com/products/clearcaster), Wowza’s live streaming platform makes the conversion and delivery process seamless.

<p align="center">
    <img src="/img/2019/Protocols-Workflow-1.png" />
</p>

[Robert Gibb at StackPath writes](https://blog.stackpath.com/rtmp/):
> "A media server is an absolute necessity if you want to leverage RTMP for live streaming. Wowza Streaming Engine, for example, is a widely used streaming software for live and on-demand video that can be installed on any server."

# Considerations When Replacing RTMP
The best [protocol](https://www.wowza.com/blog/streaming-protocols) for your workflow will depend on your use case. You can check out the pros and cons of each protocol in this blog. But to start, we’d recommend comparing your options based on the considerations below:

- Scalability
- Latency
- Quality of experience (adaptive bitrate enabled, etc.)
- Use (first-mile contribution vs. last-mile delivery)
- Playback support
- Proprietary vs. open source
- Codec requirements

Transcoding RTMP live streams into adaptive HLS and DASH formats remains a common practice — and might be the best place to start. That said, WebRTC will be better suited for anyone requiring sub-500ms latency.

# RTMP Alternatives for Egress
For last-mile delivery, Apple’s [HTTP Live Streaming (HLS)](https://www.wowza.com/blog/hls-streaming-protocol) and [MPEG-DASH](https://www.wowza.com/blog/streaming-protocols#mpeg-dash) lead the pack. HLS is the most common protocol in use for live streaming today, employed by more than 45% of participants in our [Video Streaming Latency Report](https://www.wowza.com/blog/2019-video-streaming-latency-report). MPEG-DASH is the open-source alternative to HLS and a standard in the industry.

While HTTP-based protocols have gotten heat in the past for injecting far more latency that RTMP, that’s expected to change. [Low-latency CMAF](https://www.wowza.com/blog/low-latency-cmaf-chunked-transfer-encoding) for DASH and [Low-Latency HLS](https://www.wowza.com/blog/apple-low-latency-hls) promise to support low-latency streaming at scale — and thus delivering the best of both worlds.

That said, the only [ultra-low-latency alternative to RTMP is WebRTC](https://www.wowza.com/blog/using-webrtc-as-a-replacement-for-rtmp). The WebRTC framework allows users to communicate directly through their browsers with sub-500ms latency.

In the January/February 2020 issue of Streaming Media magazine, Robert Reinhard explains, "If you’re using Flash for low-latency real-time streaming, you’ve got about a year or less to try moving over to a WebRTC solution. And what does that mean exactly? Any code you’re using on your Flash-based media server (Adobe Media Systems, Wowza Streaming Engine, and so on) needs to migrate to WebRTC instead of Real-Time Messaging Protocol (RTMP)"

While WebRTC wins the race when it comes to low-latency streaming, it isn’t without limitations. The technology was designed for one-to-few broadcasts and thus needs to be customized for large-scale deployments.


# RTMP Alternatives for Ingress
While RTMP is still commonplace for first-mile contribution, even that could change with time. Industry leaders predict that open-source protocols like [Secure Reliable Transport (SRT)](https://www.wowza.com/low-latency/SRT-secure-reliable-transport) could become the standard.

[According to Deloitte](https://www2.deloitte.com/content/dam/insights/us/articles/722835_tmt-predictions-2020/DI_TMT-Prediction-2020.pdf): 
> "Legacy protocols such as real-time messaging protocol (RTMP), developed over a decade ago to encode video and move it across networks to clients, will likely be displaced by newer solutions such as Secure Reliable Transport (SRT), designed to further decrease latency and meet the demands of live and on-demand streaming."

Just what is SRT? Wowza cofounded the [SRT Alliance](https://www.srtalliance.org/) with Haivision in 2017 as part of an ongoing effort to minimize latency when delivering live video. The protocol helps solve for video contribution and distribution challenges across suboptimal networks.

[Read more...](https://www.wowza.com/blog/rtmp-streaming-real-time-messaging-protocol)