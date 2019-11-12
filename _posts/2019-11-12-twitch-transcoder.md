---
layout: post
title: Live Video Transmuxing/Transcoding: FFmpeg vs TwitchTranscoder
image: https://blog.twitch.tv/assets/uploads/0b997d21bf8f196b6f186a5961fab806.png
tags: [FFMPEG, HLS, Codec, Twitch]
---

## Background
`Twitch` is the world’s leading live streaming platform for video games, esports, and other emerging creative content. Every month, more than 2.2 million unique content creators stream or upload video on our website. At its peak, Twitch ingests tens of thousands of concurrent live video streams and delivers them to viewers across the world.

Figure 1 depicts the architecture of our live video `Content Delivery Network` ([CDN](https://en.wikipedia.org/wiki/Content_delivery_network)) which delivers tens of thousands of concurrent live streams internationally.

<p align="center">
    <img src="https://blog.twitch.tv/assets/uploads/9406390810b8b00cd754d7d2332cfabb.png" />
    <i>Figure 1</i>
</p>

`Twitch`, like many other live streaming services, receives live stream uploads in `Real-Time Messaging Protocol` ([RTMP](https://en.wikipedia.org/wiki/Real-Time_Messaging_Protocol)) from its broadcasters. RTMP is a protocol designed to stream video and audio on the Internet, and is mainly used for point to point communication. To then scale our live stream content to countless viewers, Twitch uses `HTTP Live Streaming` ([HLS](https://en.wikipedia.org/wiki/HTTP_Live_Streaming)), an HTTP-based media streaming communications protocol that most video websites also use.

Within the live stream processing pipeline, the transcoder module is in charge of converting an incoming `RTMP` stream into the `HLS` format with multiple variants (e.g., 1080p, 720p, etc.). These variants have different bitrates so that viewers with different levels of download bandwidth are able to consume live video streams at the best possible quality for their connection.

<p align="center">
    <img src="https://blog.twitch.tv/assets/uploads/721496d4059040ba985be574e9afc577.png" />
    <i>Figure 2 depicts the input and output of the transcoder module in our live video CDN</i>
</p>

In this article, we will discuss

- How `FFmpeg` satisfies most of the live transcoding requirements,

- What features `FFmpeg` doesn’t provide, and

- Why `Twitch` builds its own in-house transcoder software stack.

## Using FFmpeg Directly

[FFmpeg](https://www.ffmpeg.org/) is a popular open-source software project, designed to record, process and stream video and audio. It is widely deployed by cloud encoding services for file [transcoding](https://en.wikipedia.org/wiki/Transcoding) and can also be used for live stream [transmuxing](https://en.wikipedia.org/wiki/Transmux) and transcoding.

Suppose we are receiving the most widely used video compression standard of [H.264](https://en.wikipedia.org/wiki/H.264/MPEG-4_AVC) in RTMP at 6mbps and 1080p60 (resolution of 1920 by 1080 with a frame rate of 60 frames per second). We want to generate 4 HLS variants of:

- 1080p60 HLS/H.264,

- 720p60 HLS/H.264,

- 720p30 HLS/H.264

- 480p30 HLS/H.264

One solution is to run 4 independent instances of FFmpeg, each of them processing one variant. Here we set all of their `Instantaneous Decoding Refresh` ([IDR](https://en.wikipedia.org/wiki/Network_Abstraction_Layer)) intervals to 2 seconds and turn off the scene change detection, thus allowing the output HLS segments of all variants are perfectly time aligned, which is required by the HLS standard.

FFmpeg 1-in-1-out Sample Commands:

```bash
ffmpeg -i <input_file> \
-c:v libx264 -x264opts keyint=:no-scenecut \
-s -r -b:v -profile:v -c:a aac \
-sws_flags -hls_list_size .m3u8
```

Notes:

All parameters surrounded by `<>` require inputs from the user. Some of these are described in greater detail below

- `c:v` specifies the video codec to use, in our case, `libx264`

- `x264opts` specifies libx264 specific options. Here, `IDR interval` should be 2 * your desired `FPS`, so `720p60` would yield an `IDR interval` of 120 while `720p30` would require an `IDR interval` of 60. No-scenecut is used to disable scene change detection

- `s` specifies video size which can be in the form of __`width x height`__ or the name of a size abbreviation

- `r` specifies the `FPS`

- `b:v` specifies a target video bitrate, which is most useful for streaming when there are bandwidth targets and requirements; there is a companion `b:a` for audio

- `profile` refers to [H.264 profile](https://en.wikipedia.org/wiki/H.264/MPEG-4_AVC#Profiles)

- `sws_flags` determines what scaling algorithm should be used

- `hls_list_size` is used to determine the maximum number of segments in the playlist (e.g., we can use 6 for live streaming or set it equal to 0 to have a playlist of all the segments). The segment duration (the optional `hls_time` flag) will be same as the `IDR interval`, in our case is 2 seconds.

Since `H.264` is a lossy compression standard, transcoding will inevitably trigger video quality degradation. Moreover, encoding is a very computationally expensive process, particularly for high resolution and high frame rate video. Due to these two constraints, we would ideally like to transmux rather than transcode the highest variant from the source RTMP to save the computational power and preserve the video quality.

In the above example, if we want to transmux an input `1080p60` RTMP source to HLS, we can actually use the above commands without specifying a size or target FPS and specifying copy for the codecs (to avoid decoding and re-encoding the source):

```bash
ffmpeg -i <input_file> -c:v copy -c:a copy -hls_list_size .m3u8
```

Transmuxing the source bitstream is an effective technique, but might cause output HLS to lose its spec compliancy, causing it to be unplayable on certain devices. We will explain the nature of the problem and its ramifications in the next section.

On the other hand, `FFmpeg` does have the functionality of taking in 1 input and producing N outputs, which we demonstrate with the following `FFmpeg` command.

_FFmpeg 1-in-N-out Sample Commands (use Main Profile, x264 veryfast preset, and the bilinear scaling algorithm):_

```bash
ffmpeg -i <input_file> \
-c:v libx264 -x264opts keyint=120:no-scenecut -s 1920x1080 -r 60 \
-b:v -profile:v main -preset veryfast -c:a libfdk_aac \
-sws_flags bilinear -hls_list_size  .m3u8 \

-c:v libx264 -x264opts keyint=120:no-scenecut -s 1280x720 -r 60 \
-b:v -profile:v main -preset veryfast -c:a libfdk_aac \
-sws_flags bilinear -hls_list_size  .m3u8 \

-c:v libx264 -x264opts keyint=60:no-scenecut -s 1280x720 -r 30 \
-b:v -profile:v main -preset veryfast -c:a libfdk_aac \
-sws_flags bilinear -hls_list_size  .m3u8 \

-c:v libx264 -x264opts keyint=60:no-scenecut -s 852x480 -r 30 \
-b:v -profile:v main -preset veryfast -c:a libfdk_aac \
-sws_flags bilinear -hls_list_size  .m3u8

```

If we wanted to transmux instead of transcode the highest variant while transcoding the rest of the variants, we can replace the the first output configuration with the previously specified copy codec:

```bash
-c:v copy -c:a copy -hls_list_size  .m3u8
```

> Notes:

> - From the above command, we are transcoding multiple variants out of a single input file. Each “\” denotes a new line, in which we can specify a different combination of flags as well as a unique output name. Each command is independent of the other and can use any other combination of flags.

> - The primary differences for each command here can be seen in the s and r flags which are explained earlier in the article.

> - An alternative to running the following transcodes in a single FFmpeg instance is to run multiple instances, one for each desired output in parallel. The 1-in-N-out FFmpeg is a computationally cheaper process, whose reason we will explain next.

## A Few Technical Issues

The previous section demonstrated how `FFmpeg` can be used to generate HLS for live streams. While useful, a few technical issues make `FFmpeg` a less-than-ideal solution.

_Transmux and Transcode_

<p align="center">
    <img src="https://blog.twitch.tv/assets/uploads/21f280d3eaf85ca46e6d0c7ba626bdf7.png" />
    <i>Figure 3</i>
</p>

In HLS, a variant is comprised of a series of segments, each starting with an IDR frame. The HLS spec requires that IDR frames of a variants’ corresponding segments be aligned, such that they will have the same `Presentation Timestamp` ([PTS](https://en.wikipedia.org/wiki/Presentation_timestamp)). Only in this way can an `HLS Adaptive Bitrate` ([ABR](https://en.wikipedia.org/wiki/Adaptive_bitrate_streaming)) player seamlessly switch among the variants when the viewer’s network condition changes (see Figure 3).

<p align="center">
    <img src="https://blog.twitch.tv/assets/uploads/85a60492f18f8ddc521ac7bf3ae7794f.png" />
    <i>Figure 4</i>
</p>

If we transcode both the source and the rest of the variants, we will get perfectly time-aligned `HLS segments` since we force the `FFmpeg` to encode IDRs precisely every 2 seconds. However, we don’t have control over the IDR intervals in the source RTMP bitstream, which is completely determined by the broadcast software’s configuration. If we transmux the source, the segments of the transmuxed and transcoded variants are not guaranteed to align (see Figure 4). This misalignment can cause playback issues. For example, we have noticed that [Chromecast](https://en.wikipedia.org/wiki/Chromecast) constantly exhibits playback pauses when it receives HLS streams with misaligned segments.

For the source RTMP stream with variable `IDR intervals`, we ideally want the output HLS to look aligned like in Figure 5:

<p align="center">
    <img src="https://blog.twitch.tv/assets/uploads/221d1a11939482fecb69acb0f33104a7.png" />
    <i>Figure 5</i>
</p>

However, in both the __1-in-1-out and the 1-in-N__-out FFmpeg instances, the N encoders associated with the N output variants are independent. As explained above, the result IDRs will not be aligned (see Figure 4) unless all variants are transcoded (i.e. the highest variant is also transcoded not transmuxed from the source).

_Software Performance_

As discussed in Figure 2, our RTMP-HLS transcoder takes in an input of 1 stream and produces an output of __N__ streams (N = the number of HLS variants. E.g., N = 4 in Figure 5). The simplest way to achieve this output is to create N independent 1-in-1-out transcoders, with each generating 1 output stream. The FFmpeg solution described above utilizes this model and has __N__ FFmpeg instances.

There are 3 components within a _1-in-1-out_ transcoder, namely [decoder](https://en.wikipedia.org/wiki/Video_decoder), [scaler](https://en.wikipedia.org/wiki/Video_scaler), and [encoder](https://en.wikipedia.org/wiki/Video_decoder) (see Figure 6). Therefore, for __N__ FFmpeg instances, we will have N decoders, N scalers, and N encoders altogether.

<p align="center">
    <img src="https://blog.twitch.tv/assets/uploads/fafdd4efb43d05e6faa42f9e3fa091c2.png" />
    <i>Figure 6</i>
</p>

Since the __N__ decoders are identical, the transcoder should ideally eliminate the redundant N-1 decoders and feed decoded images from the only decoder to the N downstream scalers and encoders (see Figure 7).

<p align="center">
    <img src="https://blog.twitch.tv/assets/uploads/862300f4e9c083ca89a1f98f9a494e37.png" />
    <i>Figure 7</i>
</p>


We talked about the decoder redundancy above. Now, let’s take another look at the four-variant example.

- 1080p60 HLS/H.264,

- 720p60 HLS/H.264,

- 720p30 HLS/H.264, and

- 480p30 HLS/H.264

The two transcoded variants `720p60` and `720p30` in this example can share one scaler. As gathered from experiments, scaling is a very computationally expensive step in the transcoding pipeline. Avoiding the unnecessary repeated scaling process altogether can significantly optimize the performance of our transcoder. Figure 8 depicts a threading model of combining the scalers of `720p60` and `720p30` variants.

<p align="center">
    <img src="https://blog.twitch.tv/assets/uploads/f49efd672a6ef252f289390de2987473.png" />
    <i>Figure 8</i>
</p>

Besides the decoder and scaler sharing, a more important feature is multithreading. Since both the encoder and scaler are very computationally expensive processes, it is critical to utilize the modern multi-core CPU architecture and let the multiple variants be processed simultaneously. From our experiment, we find multithreading very useful for achieving higher density and is critical for certain applications like 4K.

_Custom Features_

`FFmpeg` is a versatile video processing software supporting various video/audio formats for the standard ABR transcoding workflow. However, it cannot handle a number of technical requirements that are specific to Twitch’s operation. For example,

_1) Frame rate downsampler_

Our typical incoming bitstreams have 60fps (frames per second), and we transcode them to 30fps for lower bit-rate variants (e.g., 720p30, 480p30, etc.). On the other hand, since Twitch is a global platform, we often receive incoming streams of 50fps, most of which are from [PAL](https://en.wikipedia.org/wiki/PAL) countries. In this case, the lower bit-rate variant should be downsampled to 25fps, instead of 30fps.

Simply deleting every second frame is not a good solution here. Our downsampler needs to behave differently for the two different kinds of incoming bitstreams. One kind has constant frame rates less than 60fps, and the other has irregular frame dropping, which makes its average frame rates less than 60fps.

_2) Metadata insertion_

Certain information needs to be inserted into the HLS bitstreams to enhance the user experience on the player side. By building our own transcoder and player, `Twitch` can control the full end-to-end ingest-transcode-CDN-playback pipeline. This allows us to insert proprietary metadata structures into the transcoder output, which are eventually parsed by our player and utilized to produce effects unique to `Twitch`.

## FFmpeg’s 1-In-N-Out Pipeline. Why doesn’t it handle the technical issues discussed earlier?

How does FFmpeg programmatically deal with instances where a single input stream is required to generate multiple transcoded and/or transmuxed outputs? We went directly into the FFmpeg Release 3.3. [Source code](https://github.com/FFmpeg/FFmpeg/tree/release/3.3) in order to understand its threading model and transcoding pipeline.

> Note:
> The latest FFmpeg release [4.2.1](https://github.com/FFmpeg/FFmpeg/tree/release/4.2)

In the top-level `ffmpeg.c` file, the transcode() function (line `4544`) loops and repeatedly calls `transcode_step()` (line `4478`) until its inputs are completely processed, or until the user interrupts the execution. The `transcode_step()` method wraps the main pipeline and orchestrates file _I/O_, filtering, decoding and encoding amongst many other immediate steps.

During the initial setup phase, `init_input_threads()` (line `4020`) is called, and based on the number of input files, a number of new threads may be spawned to process the input.

```C
if (nb_input_files == 1) {
   return 0;
}

for (i = 0; i < nb_input_files; i++) {

   ...

   ret = av_thread_message_queue_alloc(&f->in_thread_queue, f->thread_queue_size, sizeof(AVPacket));    // line 4033

}

```

In line `4033`, we see that the number of threads spawned is solely determined by the number of inputs. This means FFmpeg will process a __1-in-N-out__ scenario using only a single thread.

__In `get_input_packet()`__ (line `4055`), __the multithreaded companion function `get_input_packet_mt()`__ (line `4047`) is only called if the number of input files is greater than one. `get_input_packet_mt()` can read input frames from a message queue in a nonblocking fashion. Otherwise, we use `av_read_frame()` (line `4072`) to read a single frame for processing at one time.

```C
#if HAVE_PTHREADS

   if (nb_input_files > 1) {
      get_input_packet_mt(f, pkt);
   }

#endif
	return av_read_frame(f->ctx, pkt);
```

Following the frame to the end of the pipeline, it enters `process_input_packet()` (line `2591`) which decodes the frame and processes it through all the applicable filters. Timestamp correction and subtitle handling also occurs in this function. Finally, prior to returning, the decoded frame is copied to each relevant output stream.

```C
for (i = 0; pkt && i < nb_output_streams; i++) {

   ...  // check constraints

   do_streamcopy(ist, ost, pkt);    // line 2756
}

```

Lastly, __`reap_filters()`__ (line 1424) is called from `transcode_step()` to loop through each output stream. The body of `reap_filters()`’s for loop collects frames ready for processing from the buffer and encodes them before muxing them into an output file.


```C
// reap_filters line 1423

for (i = 0; i < nb_output_streams; i++) { // loop through all output streams

   ...  // initialize contexts and files

   OutputStream *ost = output_streams[i];

   AVFilterContext *filter = ost->filter->filter;

   AVFrame filtered_frame = ost->filtered_frame;

   while (1) { // process the video/audio frame for one output stream

      ...  // frame is not already complete

      ret = av_buffersink_get_frame_flags(filter, filtered_frame, …);

      if (ret < 0) {

         ...  // handle errors and logs

         break;
      }

      switch (av_buffersink_get_type(filter)) {

      case AVMEDIA_TYPE_VIDEO:

         do_video_out(of, ost, filtered_frame, float_pts);

      case AVMEDIA_TYPE_AUDIO:

         do_audio_out(of, ost, filtered_frame);

      }
      ...
}
```

By following this pipeline, we can see redundancy in how these frames are handled sequentially through the context of a single thread. We can conclude that `FFmpeg` may be suboptimal in producing results using only a single thread since the _1-in-N-out_ streaming model of transcoding is most valuable to us. `FFmpeg` documentation also suggests that in our use case, it may make more sense to [launch multiple FFmpeg instances in parallel](https://trac.ffmpeg.org/wiki/Creating%20multiple%20outputs). Our key insight here is that while multithreaded functionality does exist for this tool, it does not match the exact needs of `Twitch`’s streaming service and cannot be used as we would like.

### Benchmarks

TwitchTranscoder is our in-house software developed to address the technical issues discussed earlier. It has been used in our production to process tens of thousands of concurrent live streams 24/7.

To determine if the TwitchTranscoder would perform better than FFmpeg on daily transcoding tasks, we performed a series of basic benchmark tests. For our tests, we fed both tools with a Twitch live stream as well as a 1080p60 video file using the same presets, profiles, bitrates, and other flags. Each source was transcoded to our typical stack of 720p60, 720p30, 480p30, 360p30, and 160p30.

Our hypothesis was that FFmpeg would transcode slower than TwitchTranscoder for file input, and might even fail to keep up for live streaming.

The results in figures 9, 10 and 11 compare the execution time of TwitchTranscoder vs. FFmpeg. They show that our transcoder is indeed faster for offline transcoding, even as we handled the same task and more (providing audio only transcoding, thumbnail generation, and so on in addition to the stack specified above).

FFmpeg is slightly faster for single-variant output 720p60 because TwitchTranscoder handles more tasks as explained above. When the number of variants increases, TwitchTranscoder’s multithreading model has a greater advantage that helps it outperform FFmpeg. For Twitch’s full ABR ladder, TwitchTranscoder saves 65% execution time in comparison to FFmpeg.

<p align="center">
    <img src="https://blog.twitch.tv/assets/uploads/22fab7bb307410193f25b0b3278e7a06.png" />
    <i>Figure 9</i>
</p>

<p align="center">
    <img src="https://blog.twitch.tv/assets/uploads/b1e00f69823841feeb331d4e5aa85448.png" />
    <i>Figure 10</i>
</p>

<p align="center">
    <img src="https://blog.twitch.tv/assets/uploads/b35c884634d96286af6c0502b4593aa8.png" />
    <i>Figure 11</i>
</p>

We conducted our live stream transcoding tests by comparing how many parallel instances of FFmpeg would be able to run on a single machine before issues appeared. Issues can include dropped frames, video artifacts, etc. In our production servers, we are able to support multiple channels being transcoded simultaneously while many more channels are transmuxed. Unfortunately, running more than a single FFmpeg instance causes a slew of errors that affect the transcoded outputs, and requires much more CPU utilization (see screenshot in Figure 12).

<p align="center">
    <img src="https://blog.twitch.tv/assets/uploads/6332b812cafdefbb4b475f8d6bf28b4b.png" />
    <i>Figure 12</i>
</p>

### Conclusion

In this article, we examined `FFmpeg` as a live stream `RTMP-to-HLS` transcoder and provided insight on how to operate the tool. Such a solution is simple to deploy and has a few technical issues, such as segment misalignment, unnecessary performance sacrifice, and a lack of flexibility to support our product features. Therefore, we implemented our own in-house transcoder software stack called TwitchTranscoder, which runs in a custom designed threading model and outputs _N_ variants in a single process.

Source [Twitch's blog](https://blog.twitch.tv/en/2017/10/10/live-video-transmuxing-transcoding-f-fmpeg-vs-twitch-transcoder-part-i-489c1c125f28/)

> Thanks!