---
layout: post
title: "Video Transcoding Pipeline"
tags: [FFMPEG, Video, Codec]
comments: true
---

<p align="center">
    <img src="/img/2020/pipeline.jpg" />
    <i>Figure 1</i>
</p>

- Ingest: The video being uploaded to Uiza system
- Metadata Inspect: The process of parsing all metadata information of the video like resolution, duration, framerate… for validation and making decision for later process: number of chunks to split , worker specs to handle the transcoding
- Video Splitting : the video being split into multiple chunks without the audio.
- Multiple Chunks Encoding: each chunk will be transcoded into multiple profiles by one worker. The audio is transcoded directly from the video source .
- Video Validation : validate all transcoded chunks after transcoding process are completed . If there’s missing or corrupted chunks, the transcoding process will be restart for said chunk.
- Video Assembler: all transcoded chunks will be stitched together, profile by profile and then package into HLS / DASH asset .
- Upload to Origin: HLS/DASH asset will be uploaded to Uia Origin Storage.

<p align="center">
    <img src="/img/2020/vodv5.jpg" />
    <i>Figure 2</i>
</p>


- `Validation module`: The process of parsing all metadata information of the video like resolution, duration, framerate… for validation and making decision for later process: number of chunks to split, worker specs to handle the transcoding.
- `Splitter module` : the video being split into multiple chunks video without the audio and encode audio.
- `Encoder module`: each chunk will be transcoded into multiple profiles by one worker.
- `Assembler module`: validate all transcoded chunks and all transcoded chunks will be stitched together, profile by profile and then package into `HLS` / `DASH` asset.

# 1. Validation module

<p align="center">
    <img src="/img/2020/validation_module.png" />
    <i>Figure 1</i>
</p>

Currently we only support reading metadata from `S3` / `HTTP` / `HTTPS`.

__Required Metadata__

Currently, we only support 1 video/audio stream metdata set per file . For multiple video/audio stream in one file, we will enhance this process when we support multiple stream encoding.

In ffprobe output, each stream have index number and `codec_type` param. We will validate the first video and audio stream only (check `codec_type`, it only have `video/audio/data` value) 

Currently, we only support user input file extension: `mp4, mov, mkv, m4v, avi, ts, mpg, webm, wmv, flv`


<table>
    <thead>
        <tr>
            <th>#</th>
            <th>metadata</th>
            <th>example value</th>
            <th>description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan=7>FORMAT</td>
            <td rowspan=7>format_name</td>
            <td>• mov, mp4, m4a, 3gp, 3g2, mj2</td>
            <td>• mov, mp4, m4a, 3gp, 3g2, mj2 (.mov, .mp4, .m4v)</td>
        </tr>
        <tr>
            <td>• avi</td>
            <td>• avi (.avi)</td>
        </tr>
        <tr>
            <td>• asf</td>
            <td>• asf (.wmv)</td>
        </tr>
        <tr>
            <td>• matroska,webm</td>
            <td>• matroska,webm (.mkv)</td>
        </tr>
        <tr>
            <td>• mpegts</td>
            <td>• mpegts (.ts)</td>
        </tr>
        <tr>
            <td>• mpegvideo</td>
            <td>• mpegvideo (.mpg)</td>
        </tr>
        <tr>
            <td>• flv</td>
            <td>• flv</td>
        </tr>
        <tr>
            <td rowspan=3>VIDEO</td>
            <td>width</td>
            <td>1920...</td>
            <td>Width of video</td>
        </tr>
        <tr>
            <td>height</td>
            <td>1080...</td>
            <td>Height of video</td>
        </tr>
        <tr>
            <td>r_frame_rate</td>
            <td>25/1...</td>
            <td>Video framerate</td>
        </tr>
        <tr>
            <td rowspan=2>AUDIO</td>
            <td>sample_rate</td>
            <td>44100...</td>
            <td>Audio Samplerate</td>
        </tr>
        <tr>
            <td>channels</td>
            <td>2...</td>
            <td>Number of audio channels</td>
        </tr>
    </tbody>
</table>


__Rules of validation__

- If ffprobe can not read the file and output to `json` format---->Return error code as defined
- If ffprobe output missing any required metadata above, or one of the metadata has null value --->Return error code as defined, with error_message=”missing required metadata: video_duration” (for example, that video_duration is missing or has null value)

Payload
```json
{
  "id": "923d9a0df8affe69",
  "entity_id": "7jfeu-rewfre-31341-3414311",
  "media_file": {
    "url": "s3://uiza-ulas-test.s3.ap-southeast-1.amazonaws.com/input.mp4",
    "metadata": {}
  }
}
```

Result

```json
{
    "id": "923d9a0df8affe69",
    "entity_id": "7jfeu-rewfre-31341-3414311",
    "media_file": {
        "url": "s3://uiza-ulas-test.s3.ap-southeast-1.amazonaws.com/input.mp4",
        "metadata": {
            "audios": [
                {
                    "codec_long_name": null,
                    "sample_format": "fltp",
                    "duration": 10.005333,
                    "channel_layout": "stereo",
                    "type": "aac",
                    "default": "1",
                    "stream": 1,
                    "channels": 2,
                    "bit_rate": 128,
                    "sample_rate": 48000,
                    "format": "aac"
                }
            ],
            "videos": [
                {
                    "aspect_ratio": null,
                    "frame_rate": 23,
                    "height": 240,
                    "width": 416,
                    "bit_rate": 383,
                    "stream": 0,
                    "pix_format": null,
                    "duration": 10.00834,
                    "codec_name": "h264",
                    "profile": "High",
                    "rotate": 0
                }
            ]
        }
    },
    "response_status": {
        "code": "",
        "message": ""
    }
}
```

# 2. Splitter Module

<p align="center">
    <img src="/img/2020/splitter.png" />
    <i>Figure 1</i>
</p>

__Logic__

<table>
    <thead>
        <tr>
            <th>#</th>
            <th>Splitter factor</th>
            <th>Max duration per segment(mins)</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan=3>Input Resolution</td>
            <td>SD (W x H < 921600)</td>
            <td>4</td>
        </tr>
        <tr>
            <td>HD (921600 <= WxH <= 2073600)</td>
            <td>2</td>
        </tr>
        <tr>
            <td>UHD (2073600 < WxH <= 8294400)</td>
            <td>1</td>
        </tr>
    </tbody>
</table>


Base on the resolution metadata we store of the input video source in Validation step, we will define the max duration per segment when splitting the video. 

> Note: The logic will be expanded later on.

__Worker__

The worker will read the input source from storage and execute 2 parallel process: one is splitting the video track into segments with `MKV` format, the other is transcoding the audio file into `AAC 128Kbps` with `M4A` format. Both will be transfer to storage on the fly.

Example Commandline:

```bash
ffmpeg -i input_file -map 0:0 -c:v copy -f segment -segment_time 60 \
-y Downloads/video_%d.mkv -map 0:1 -c:a aac -b:a 128K \
-af “aresample=async=1:min_hard_comp=0.100000:first_pts=0” -y audio.m4a
```

Note:

- Assume that the input file has stream `0` for video, and stream `1` for audio, hence the mapping `0:0` is for video and `0:1` for audio. If some how the stream index is different , please use it correctly from the ffprobe metatada.
- `Segment_time` value is seconds: `240` for `SD`, `120` for `HD`, `60` for `4K`.

__Payload__

```json
{
    "id": "923d9a0df8affe69",
    "input": {
        "media_file": {
            "url": "s3://uiza-ulas-test.s3.ap-southeast-1.amazonaws.com/input.mp4",
            "metadata": {
                "audios": [
                    {
                        "codec_long_name": null,
                        "sample_format": "fltp",
                        "duration": 10.005333,
                        "channel_layout": "stereo",
                        "type": "aac",
                        "default": "1",
                        "stream": 1,
                        "channels": 2,
                        "bit_rate": 128,
                        "sample_rate": 48000,
                        "format": "aac"
                    }
                ],
                "videos": [
                    {
                        "aspect_ratio": null,
                        "frame_rate": 23,
                        "height": 240,
                        "width": 416,
                        "bit_rate": 383,
                        "stream": 0,
                        "pix_format": null,
                        "duration": 10.00834,
                        "codec_name": "h264",
                        "profile": "High",
                        "rotate": 0
                    }
                ]
            }
        },
        "preset": {}
    }
}
```

__Result__

```json
{
    "id": "923d9a0df8affe69",
    "media_parts": {
        "videos": [
            {
                "part": 0,
                "media_file": {
                    "url": "s3://uiza-ulas-test.s3.ap-southeast-1.amazonaws.com/vui_worker/923d9a0df8affe69/dev-vui-splitter/input_0.mkv",
                    "metadata": {}
                }
            }
        ],
        "audios": [
            {
                "url": "s3://uiza-ulas-test.s3.ap-southeast-1.amazonaws.com/vui_worker/923d9a0df8affe69/dev-vui-splitter/audio.m4a",
                "metadata": {}
            }
        ]
    },
    "response_status": {
        "code": "",
        "message": ""
    }
}
```

# 3. Encoder module

__Format__

```bash
ffmpeg -i INPUT -filter_complex "[v:0]split=2[s0][s1];[s0]scale=trunc(oh/2*dar)*2:OW0,setsar=1[v0];[s1]scale=trunc(oh/2*dar)*2:OW1,setsar=1[v1]" \
-map "[v0]" -c:v:0 OUTPUT_CODEC -r FRAMERATE -g SEG_DURATION*FRAMERATE \
-sc_threshold 0 -profile:v:0 VIDEO_PROFILE -preset fast -crf 23 \
-rc-lookahead 32 -b:v:0 BITRATE -bufsize:v:0 BITRATE*1.5 \
-maxrate:v:0 BITRATE*1.5 -color_primaries 1 -color_trc 1 -colorspace 1 \
-movflags +faststart -y OUTPUT_0.mp4 -map "[v1]" -c:v:1 OUTPUT_CODEC \
-r FRAMERATE -g SEG_DURATION*FRAMERATE -sc_threshold 0 \
-profile:v:1 VIDEO_PROFILE -preset fast -crf 23 -rc-lookahead 32 \
-b:v:1 BITRATE -bufsize:v:1 BITRATE*1.5 -maxrate:v:1 BITRATE*1.5 \
-color_primaries 1 -color_trc 1 -colorspace 1 \
-movflags +faststart -y OUTPUT_1.mp4
```

<table>
    <thead>
        <tr>
            <th>Value</th>
            <th>Description</th>
            <th>Example</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>INPUT</td>
            <td>path to video segment from splitting process</td>
            <td></td>
        </tr>
        <tr>
            <td>OW</td>
            <td>Output height from Video Preset</td>
            <td>1080, 720,480,360..</td>
        </tr>
        <tr>
            <td>OUTPUT_CODEC</td>
            <td>Video codec from Video Preset</td>
            <td>libx264(H264), libx265 (H265)</td>
        </tr>
        <tr>
            <td>FRAMERATE</td>
            <td>Framerate from Video Preset</td>
            <td>30,60,120….</td>
        </tr>
        <tr>
            <td>SEG_DURATION</td>
            <td>Segment Duration from Video Preset</td>
            <td>2,4,10</td>
        </tr>
        <tr>
            <td>VIDEO_PROFILE</td>
            <td>Base on output resolution</td>
            <td>Main(SD), High (HD, 4K)</td>
        </tr>
        <tr>
            <td>BITRATE</td>
            <td>Bitrate from Video Preset</td>
            <td>800K, 1200K…</td>
        </tr>
    </tbody>
</table>

__Example__

Encoding using preset that contain 2 profiles `HD` & `SD` 

```bash
ffmpeg -i input.mp4 -filter_complex "[v:0]split=2[s0][s1];[s0]scale=trunc(oh/2*dar)*2:720,setsar=1[v0];[s1]scale=trunc(oh/2*dar)*2:480,setsar=1[v1]" \
-map "[v0]" -c:v:0 libx264 -r 30 -g 60 -sc_threshold 0 -profile:v:0 high \
-preset fast -crf 23 -rc-lookahead 32 -b:v:0 1500K -bufsize:v:0 2250K \
-maxrate:v:0 2250K -color_primaries 1 -color_trc 1 -colorspace 1 \
-movflags +faststart -y output0.mp4 -map "[v1]" -c:v:1 libx264 -r 30 \
-g 60 -sc_threshold 0 -profile:v:1 main -preset fast -crf 23 \
-rc-lookahead 32 -b:v:1 400K -bufsize:v:1 600K -maxrate:v:1 600K \
-color_primaries 1 -color_trc 1 -colorspace 1 \
-movflags +faststart -y output1.mp4
```

__Payload__

```json
{
    "id": "923d9a0df8affe69",
    "part": 0,
    "media_file": {
        "url": "s3://uiza-ulas-test.s3.ap-southeast-1.amazonaws.com/vui_worker/923d9a0df8affe69/dev-vui-splitter/input_0.mkv",
        "metadata": {
            "audios": [
                {
                    "codec_long_name": null,
                    "sample_format": "fltp",
                    "duration": 10.005333,
                    "channel_layout": "stereo",
                    "type": "aac",
                    "default": "1",
                    "stream": 1,
                    "channels": 2,
                    "bit_rate": 128,
                    "sample_rate": 48000,
                    "format": "aac"
                }
            ],
            "videos": [
                {
                    "aspect_ratio": null,
                    "frame_rate": 23,
                    "height": 240,
                    "width": 416,
                    "bit_rate": 383,
                    "stream": 0,
                    "pix_format": null,
                    "duration": 10.00834,
                    "codec_name": "h264",
                    "profile": "High",
                    "rotate": 0
                }
            ]
        }
    },
    "preset": {
        "id": "3693d866-1577-48f4-a97c-6e15765ba0fe",
        "name": "Uiza LIVE 480p",
        "description": "",
        "is_default": true,
        "profile": [
            {
                "name": "360",
                "orientation": "landscape",
                "video_profile": {
                    "video_framerate": 30,
                    "video_resolution": 360,
                    "video_codec": "libx264",
                    "video_bitrate": "700K"
                },
                "audio_profile": {
                    "audio_bitrate": "128K",
                    "audio_codec": "aac",
                    "audio_channel": 2
                },
                "segment_duration": 10
            },
            {
                "name": "720",
                "orientation": "landscape",
                "video_profile": {
                    "video_framerate": 30,
                    "video_resolution": 720,
                    "video_codec": "libx264",
                    "video_bitrate": "1400K"
                },
                "audio_profile": {
                    "audio_bitrate": "128K",
                    "audio_codec": "aac",
                    "audio_channel": 2
                },
                "segment_duration": 10
            }
        ]
    }
}
```

__Result__

```json
{
    "id": "923d9a0df8affe69",
    "part": 0,
    "encoder_output_files": [
        {
            "profile_id": "735299b7-5988-11ea-883b-028fc5698662",
            "media_file": {
                "url": "s3://uiza-ulas-test.s3.ap-southeast-1.amazonaws.com/vui_worker/923d9a0df8affe69_0/dev-vui-encoder/360/input_0.mp4",
                "metadata": {}
            }
        }
    ],
    "response_status": {
        "code": "",
        "message": ""
    }
}
```

# 4. Assembler module

<p align="center">
    <img src="/img/2020/assembler.png" />
    <i>Figure 1</i>
</p>

__Validation__

The process of validate all encoded segments and encoded audio before assembling. The logic of validating is the number of encoded segments/encoded audio that matching with the source file.

Example: if the 12 minutes HD source file has 1 audio track, using 3 profiles to encode, and splitting into 2 minutes per segment, then the number of encoded segment is `6x3=18` and the number of encoded audio is 1.

__Assembler + Packaging:__

After valdiation process is passed, we will create a concat text file for each profile as an input. The format is:

```bash
file '/path/to/file1.mp4'
file '/path/to/file2.mp4'
file '/path/to/file3.mp4'
```

The commandline format for packaging is:

```bash
ffmpeg -f concat -safe 0 -protocol_whitelist file,s3,tls,http,tcp \
-i path/to/profile1.txt -i path/to/profile2.txt \
-i path/to/encoded_audio.m4a -map 0:v -map 1:v -map 2:a -c copy \
-movflags frag_keyframe+empty_moov -f dash \
-seg_duration PROFILE_SEGMENT_DURATION -hls_playlist 1 \
-single_file 1 -y path/to/manifest.mpd
```

__Payload:__

```json
{
    "id": "923d9a0df8affe69",
    "input": {
        "media_file": {
            "url": "s3://uiza-ulas-test.s3.ap-southeast-1.amazonaws.com/input.mp4",
            "metadata": {
                "audios": [
                    {
                        "codec_long_name": null,
                        "sample_format": "fltp",
                        "duration": 10.005333,
                        "channel_layout": "stereo",
                        "type": "aac",
                        "default": "1",
                        "stream": 1,
                        "channels": 2,
                        "bit_rate": 128,
                        "sample_rate": 48000,
                        "format": "aac"
                    }
                ],
                "videos": [
                    {
                        "aspect_ratio": null,
                        "frame_rate": 23,
                        "height": 240,
                        "width": 416,
                        "bit_rate": 383,
                        "stream": 0,
                        "pix_format": null,
                        "duration": 10.00834,
                        "codec_name": "h264",
                        "profile": "High",
                        "rotate": 0
                    }
                ]
            }
        },
        "preset": {}
    },
    "media_parts": {
        "videos": [
            {
                "part": 0,
                "media_file": {
                        "url": "s3://uiza-ulas-test.s3.ap-southeast-1.amazonaws.com/vui_worker/923d9a0df8affe69/dev-vui-splitter/input_0.mkv",
                        "metadata": {}
                },
                "encoder_output_files": [
                    {
                        "profile_id": "735299b7-5988-11ea-883b-028fc5698662",
                        "media_file": {
                            "url": "s3://uiza-ulas-test.s3.ap-southeast-1.amazonaws.com/vui_worker/923d9a0df8affe69_0/dev-vui-encoder/360/input_0.mp4",
                            "metadata": {}
                        }
                    }
                ]
            }
        ],
        "audios": [
            {
                "url": "s3://uiza-ulas-test.s3.ap-southeast-1.amazonaws.com/vui_worker/923d9a0df8affe69/dev-vui-splitter/audio.m4a",
                "metadata": {}
            }
        ]
    }
}
```

__Result__

```json
{
    "id": "923d9a0df8affe69",
    "output": [
        {
            "type": "dash",
            "media_file": {
                "url": "s3://uiza-ulas-test.s3.ap-southeast-1.amazonaws.com/vui_worker/923d9a0df8affe69/dev-vui-splitter/input_0.mkv",
                "metadata": {}
            }
        },
        {
            "type": "hls",
            "media_file": {
                "url": "s3://uiza-ulas-test.s3.ap-southeast-1.amazonaws.com/vui_worker/923d9a0df8affe69/dev-vui-splitter/input_0.mkv",
                "metadata": {}
            }
        }
    ],
    "response_status": {
        "code": "",
        "message": ""
    }
}
```