# Hikvision RTSP Video and Audio in Home Assistant

Getting video from a Hikvision camera is usually the easy part.

Getting the **right stream, the expected resolution, and working audio** can take a little more experimentation — especially with Chinese-market cameras and different firmware versions.

This page documents the approach that worked in our Home Assistant installation.

## RTSP stream URLs

Hikvision cameras commonly expose RTSP streams using this pattern:

```text
rtsp://USERNAME:PASSWORD@CAMERA_IP:554/Streaming/Channels/101
```

For channel 1, the most common stream IDs are:

```text
101 = main stream
102 = sub stream
```

Example with placeholders:

```text
rtsp://USERNAME:PASSWORD@192.168.1.101:554/Streaming/Channels/101
```

Never publish real camera credentials in a public repository.

## Main stream vs sub stream

The two streams are useful for different jobs.

The **main stream** normally provides the best image quality and is the stream we use when creating security snapshots.

The **sub stream** is lower bandwidth and may be more convenient for dashboards, remote viewing, or devices where decoding the full main stream is unnecessary.

Do not assume the configured resolution from the camera web interface is the resolution Home Assistant is actually receiving. Verify the stream directly.

## Check the stream with FFmpeg

A useful diagnostic command is:

```bash
ffmpeg -rtsp_transport tcp \
  -i "rtsp://USERNAME:PASSWORD@192.168.1.101:554/Streaming/Channels/101"
```

FFmpeg will print information about the streams it detects.

In our tested setup, the main Hikvision stream was detected as HEVC/H.265 video at **2560×1440**.

The output may also show an audio stream if audio is enabled on the camera.

Press `q` to stop FFmpeg.

## Audio starts at the camera

If video works but there is no audio, first verify the camera configuration itself.

The exact menu names vary by firmware and region, but the important point is that the stream must be configured to include audio.

On Hikvision firmware this is commonly a stream setting equivalent to:

```text
Video & Audio
```

rather than:

```text
Video
```

If the camera sends video only, Home Assistant cannot recover audio that is not present in the RTSP stream.

## Verify that RTSP really contains audio

Do not troubleshoot Home Assistant first.

Check the source stream.

Run:

```bash
ffmpeg -rtsp_transport tcp \
  -i "rtsp://USERNAME:PASSWORD@192.168.1.101:554/Streaming/Channels/101"
```

Look for both a video and an audio stream in the FFmpeg output.

A simplified example might look like:

```text
Stream #0:0: Video: hevc ...
Stream #0:1: Audio: aac ...
```

The exact codec depends on the camera configuration and firmware.

If FFmpeg sees audio, the camera is delivering it and the remaining problem is somewhere later in the playback chain.

## The codec matters

One of the practical lessons from this installation was that having a microphone enabled does not automatically mean every client will play the resulting audio.

Different Hikvision models and firmware can offer codecs such as:

- G.711
- G.722.1
- G.726
- MP2L2
- AAC

Client compatibility varies.

For our setup, configuring the camera audio as **AAC at 64 kbps** restored audio in the playback path we were using.

That does not mean AAC 64 kbps is mandatory for every Hikvision/Home Assistant installation. It is simply the configuration that solved the problem on the cameras tested for this project.

## Why this can be confusing

A typical troubleshooting path looks like this:

```text
Camera microphone
       │
       ▼
Camera audio settings
       │
       ▼
RTSP stream
       │
       ▼
Home Assistant / media pipeline
       │
       ▼
Browser / mobile app / smart display
       │
       ▼
Speakers
```

Audio can fail at any point.

This is why checking the RTSP stream directly with FFmpeg is so useful: it separates a **camera configuration problem** from a **Home Assistant or playback problem**.

## Recommended troubleshooting order

If you have picture but no sound, check these in order:

1. Confirm that the camera actually has a microphone and that it is enabled.
2. Configure the camera stream for **Video & Audio**, not video-only.
3. Check the selected audio codec.
4. Verify with FFmpeg that the RTSP stream contains an audio track.
5. Test the RTSP stream with another known-good player if necessary.
6. Only then troubleshoot the Home Assistant playback path.

Changing several settings at once makes it much harder to determine which one fixed the problem.

## RTSP over TCP

Throughout this project we normally use:

```text
-rtsp_transport tcp
```

For example:

```bash
ffmpeg -rtsp_transport tcp \
  -i "rtsp://USERNAME:PASSWORD@192.168.1.101:554/Streaming/Channels/101"
```

TCP has been reliable in our local network and is also used by our snapshot commands.

## Security

RTSP URLs can contain the camera username and password directly in the URL.

Treat them as secrets.

Do not paste a real RTSP URL into:

- a public GitHub repository
- screenshots intended for publication
- forum posts
- public Home Assistant configuration examples

For published examples, use placeholders:

```text
rtsp://USERNAME:PASSWORD@192.168.1.101:554/Streaming/Channels/101
```

For your own Home Assistant configuration, consider using `secrets.yaml` or another appropriate secret-management approach instead of unnecessarily duplicating credentials.

## Chinese-market Hikvision cameras

The cameras used for this project were Chinese-market Hikvision units.

Menu names, available codecs, firmware behavior, and ISAPI capabilities may differ from international Hikvision models.

That is one reason this repository documents **observed and tested behavior** rather than assuming every Hikvision camera behaves identically.

## Related documentation

For high-resolution JPEG capture from the RTSP main stream, see:

[Full-Resolution Hikvision Snapshots](snapshots.md)

and the ready-to-adapt configuration:

[examples/snapshots.yaml](../examples/snapshots.yaml)

---

**Next:** controlling the built-in Hikvision audible alarm from Home Assistant through ISAPI.
