# Hikvision Cameras + Home Assistant Security

Turn inexpensive Chinese-market Hikvision IP cameras into a more capable Home Assistant security system.

This project documents a real Home Assistant setup built around Hikvision cameras purchased from the Chinese market. What started as a simple camera integration gradually became a distributed home-security setup with live video, audio, high-resolution snapshots, door and gate sensors, Telegram notifications, and camera-built-in alarms triggered directly from Home Assistant.

> **Work in progress:** this repository is being built from a working installation. Configuration examples and technical notes will be added step by step.

## What we got working

- Hikvision cameras integrated with Home Assistant
- Main RTSP stream instead of relying only on low-resolution snapshots
- Camera audio available in the monitoring setup
- Full-resolution JPEG snapshots captured directly from RTSP with FFmpeg
- Separate snapshot folders for each camera
- Automatic cleanup of old snapshot files
- Motion detection exposed to Home Assistant
- Door and gate sensors linked to a global Security Mode
- Telegram notifications for security events
- Armed-state check for doors or gates left open
- Hikvision ISAPI control of the cameras' built-in audible alarm
- Multiple cameras triggered together as a distributed outdoor siren system

## The idea

The interesting part is not any single feature, but how the pieces work together:

```text
Door / Gate Sensors ───────┐
                           │
Camera Motion Detection ───┼──> Home Assistant
                           │          │
Security Mode ─────────────┘          ├──> Telegram notifications
                                      ├──> FFmpeg snapshots
                                      └──> Hikvision ISAPI
                                                │
                                                ├──> Camera 1 alarm
                                                ├──> Camera 2 alarm
                                                ├──> Camera 3 alarm
                                                └──> Camera 4 alarm
```

When Security Mode is enabled, Home Assistant monitors the configured entry points. A security event can generate a notification and trigger the built-in alarms of several Hikvision cameras at the same time.

In other words, the cameras are not only cameras anymore — they also become part of the alarm system.

## Why FFmpeg snapshots?

One of the first problems was snapshot quality.

The snapshot exposed through the normal camera integration was much lower resolution than the camera's main video stream. Instead of accepting that limitation, Home Assistant captures a single frame directly from the Hikvision RTSP main stream using FFmpeg.

Example:

```yaml
shell_command:
  hikvision_camera_snapshot: >
    ffmpeg -y -rtsp_transport tcp
    -i "rtsp://USERNAME:PASSWORD@192.168.1.101:554/Streaming/Channels/101"
    -frames:v 1
    "/media/cameras/camera_1/camera_{{ now().strftime('%Y%m%d_%H%M%S') }}.jpg"
```

This gives us snapshots based on the actual main stream and also gives full control over where and how the files are stored.

## Hikvision ISAPI alarm control

Another useful discovery was that compatible Hikvision cameras with a built-in audible warning can be controlled through Hikvision's ISAPI interface.

That means Home Assistant can trigger the camera alarm as part of an automation.

In this installation, several cameras are triggered together. This effectively creates a distributed siren around the house without requiring a separate outdoor siren at every location.

The exact ISAPI calls and Home Assistant configuration will be documented in the `docs/` and `examples/` sections of this repository.

## Planned documentation

The repository will include:

```text
examples/
  snapshots.yaml
  security-mode.yaml
  door-notifications.yaml
  camera-sirens.yaml

docs/
  rtsp-and-audio.md
  snapshots.md
  hikvision-isapi.md
```

We will also document the practical issues encountered along the way, including snapshot resolution, RTSP streams, audio configuration, event handling, and alarm triggering.

## Hardware

The setup uses Chinese-market Hikvision IP cameras with features including a built-in microphone and audible/visual warning capabilities.

Exact camera model information and compatibility notes will be added as the documentation is completed.

## Security notes

Never publish real camera credentials, Telegram bot tokens, public URLs, or other secrets in a Git repository.

All examples in this repository use placeholder credentials and example IP addresses:

```text
USERNAME
PASSWORD
192.168.1.x
```

If you adapt the examples, keep your real credentials in your private Home Assistant configuration or secrets management rather than committing them to Git.

Also consider placing IP cameras on an isolated VLAN or otherwise restricting their network access where practical.

## Compatibility

This is not intended to imply that every Hikvision model exposes exactly the same features or ISAPI endpoints.

Availability of audio, warning lights, speakers, alarm functions, codecs, and ISAPI commands depends on the camera model and firmware. Test commands on your own hardware before incorporating them into security automations.

## About this project

This repository grew out of experiments with a real Home Assistant installation and Chinese-market Hikvision hardware.

The goal is to publish the parts that actually worked, including the failures and workarounds, rather than present an idealized configuration that has never been tested.

If you are working on a similar Hikvision + Home Assistant setup, feel free to use the examples as a starting point and adapt them to your cameras and network.

---

**Next:** full-resolution snapshots, RTSP/audio configuration, Security Mode automations, and Hikvision ISAPI alarm commands.
