---
name: motion-graphics-export
description: Professional export pipeline for motion graphics videos (After Effects, Remotion, or ffmpeg). Use when rendering, exporting, encoding, or delivering a motion graphics clip, explainer, promo, or social video, or when asked for a broadcast/web-ready master.
---

# Motion Graphics Export

Produce a clean master first, then derive every deliverable from it. Never re-encode a delivery file from another delivery file.

## 1. Lock the spec before rendering

Confirm these with the user if they are not stated. Use the defaults in brackets when they are not.

- Destination: web/YouTube [default], social vertical (9:16), social square (1:1), broadcast, or a web embed.
- Resolution: 1920x1080 [default] or 3840x2160 for 4K. Vertical: 1080x1920.
- Frame rate: 25 fps for PAL-region broadcast, 30 fps [default] for web, 24 fps for cinematic. Match the source frame rate of any footage; never mix rates in one timeline.
- Duration and aspect ratio. Keep a title-safe area of 90% and an action-safe area of 95% of the frame.
- Audio: stereo 48 kHz [default]. Loudness target: -14 LUFS integrated, true peak at or below -1 dBTP for web. Use -23 LUFS for EBU R128 broadcast.

## 2. Render the master

Master file: ProRes 422 HQ (`.mov`) or DNxHR HQ, same resolution and frame rate as the project, pixel format 10-bit where the codec allows. Lossless PNG sequences are acceptable for heavy effects but convert them to a single master before delivering.

Before rendering:

- Set the working colour space and keep it consistent. For web delivery, use sRGB and Rec.709 with the transfer function set to match the display target.
- Enable motion blur only where it helps; it multiplies render time.
- Collapse or pre-compose heavy layers and check for missing fonts, linked footage, and offline media.
- Use the project's full resolution and quality setting for the master, not draft.

## 3. Derive the deliverables

H.264 for maximum compatibility, H.265 when file size matters and the target supports it.

Web and YouTube (1080p, 30 fps):

```bash
ffmpeg -i master.mov -c:v libx264 -preset slow -crf 18 -profile:v high -pix_fmt yuv420p \
  -movflags +faststart -c:a aac -b:a 320k -ar 48000 -af loudnorm=I=-14:TP=-1:LRA=11 \
  -r 30 out-web-1080p.mp4
```

Vertical social (9:16): scale and pad from the master, keep text inside the safe zone:

```bash
ffmpeg -i master.mov -vf "scale=1080:1920:force_original_aspect_ratio=increase,crop=1080:1920" \
  -c:v libx264 -preset slow -crf 20 -pix_fmt yuv420p -movflags +faststart \
  -c:a aac -b:a 192k -af loudnorm=I=-14:TP=-1:LRA=11 out-vertical.mp4
```

Animated preview or poster frame: export a still at the hero frame as PNG, and a short GIF or WebM only if the page needs it.

Always use `-movflags +faststart` for web files so playback starts before the download completes.

## 4. Verify before handing over

Run these checks on every deliverable and report the results:

- `ffprobe -v error -show_entries stream=codec_name,width,height,r_frame_rate,pix_fmt -show_entries format=duration,size -of default=nw=1 out.mp4` matches the spec.
- Loudness: `ffmpeg -i out.mp4 -af ebur128=peak=true -f null -` and confirm integrated loudness and true peak are within target.
- Play the first frame, a mid-transition frame, and the last frame. Check for banding in gradients, flicker, dropped frames, and clipped text.
- Confirm the audio is in sync at the first and last cue.
- Confirm text stays inside the safe area in every aspect ratio.

## 5. Naming and delivery

Use `project-name_deliverable_WIDTHxHEIGHT_FPSfps_vN.ext`, for example `brand-intro_web_1920x1080_30fps_v3.mp4`. Keep the master and the project file next to the deliverables. Note the spec used for each file in the handoff message.

## Rules

- Do not upscale a low-resolution source and call it a master. Flag it instead.
- Do not add a watermark, logo, or credit that was not in the brief.
- If a render or encode fails, report the exact error output and the step that failed. Do not skip a verification step to finish faster.
