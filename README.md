# Sony BDP-S1700/K USB Movie Fix Guide

This guide explains how to convert movie files so they play properly from a USB flash drive on a **Sony BDP-S1700/K Blu-ray player**.

It covers two common problems:

1. **1080p video does not play from USB**
2. **Subtitles need to be transferred from the original MKV**

## Prerequisites

Before starting, make sure you have the following:

- The original movie file, usually `.mkv`
- FFmpeg installed and available from PowerShell
- FFprobe available — included with FFmpeg
- A USB flash drive formatted in a format your Sony player can read
- Optional: VLC Media Player for testing playback and subtitles on the PC

### Install FFmpeg on Windows

Open PowerShell and run:

```powershell
winget install Gyan.FFmpeg

Reload your IDE and check if properly installed:

```powershell
ffmpeg -version

---

## Target Format for Sony USB Playback

For best compatibility, convert the movie to:

```text
Container: MP4
Video: H.264 / AVC
Audio: AAC
Resolution: 1080p
Subtitles: external .srt, embedded, or burned in if needed
```

Problematic formats for this player can include:

```text
AV1 video
HEVC / H.265 video
Opus audio
10-bit video
Unsupported MKV structures
Unsupported subtitle formats
```

---

# Section 1 — Convert the Movie to Sony-Compatible 1080p MP4

## Step 1 — Open PowerShell in the Movie Folder

Open the folder where the movie file is located.

In File Explorer, click the address bar, type:

```text
powershell
```

Then press **Enter**.

This opens PowerShell directly inside that folder.

---

## Step 2 — Confirm the MKV File Is There

Run:

```powershell
Get-ChildItem -Filter *.mkv | Select-Object Name,Length
```

This confirms that PowerShell is in the correct folder and can see your movie file.

---

## Step 3 — Check the File Codecs

Replace the filename with your actual movie file name:

```powershell
ffprobe -v error -show_entries stream=index,codec_type,codec_name:stream_tags=language,title -of default=noprint_wrappers=1 "input.mkv"
```

Example problematic output:

```text
index=0
codec_name=av1
codec_type=video

index=1
codec_name=opus
codec_type=audio
```

For the Sony BDP-S1700/K, this is a problem because:

```text
AV1 video = unsupported
Opus audio = unsupported or unreliable
```

---

## Step 4 — Convert Video and Audio

Use this command:

```powershell
ffmpeg -i "input.mkv" -c:v libx264 -preset slow -crf 18 -profile:v high -level:v 4.1 -pix_fmt yuv420p -c:a aac -b:a 256k -movflags +faststart "movie-fixed.mp4"
```

What this does:

```text
-c:v libx264          Converts video to H.264 / AVC
-preset slow          Better compression/quality, slower conversion
-crf 18               Very good quality
-profile:v high       Compatible H.264 profile
-level:v 4.1          Safe level for 1080p playback
-pix_fmt yuv420p      Compatible pixel format for players/TVs
-c:a aac              Converts audio to AAC
-b:a 256k             Good stereo audio bitrate
-movflags +faststart  Improves MP4 compatibility
```

This can take a long time. For a full movie, it may take close to an hour or more depending on your PC.

---

## Step 5 — Faster Alternative for Future Movies

For a faster conversion with very similar visible quality, use `medium` instead of `slow`:

```powershell
ffmpeg -i "input.mkv" -c:v libx264 -preset medium -crf 18 -profile:v high -level:v 4.1 -pix_fmt yuv420p -c:a aac -b:a 256k -movflags +faststart "movie-fixed.mp4"
```

Quick rule:

```text
preset slow   = slower, slightly better compression
preset medium = faster, still very good quality
```

---

## Step 6 — Verify the Converted File

After conversion finishes, run:

```powershell
ffprobe -v error -show_entries stream=index,codec_type,codec_name:stream_tags=language,title -of default=noprint_wrappers=1 "movie-fixed.mp4"
```

You want to see something like:

```text
codec_name=h264
codec_type=video

codec_name=aac
codec_type=audio
```

That means the video/audio are now Sony-friendly.

---

## Step 7 — Test on the Sony Player

Copy this file to the USB flash drive:

```text
movie-fixed.mp4
```

Test it on the Sony player.

If it plays, the 1080p USB playback issue is fixed.

---

# Section 2 — Transfer Embedded Subtitles from the Original MKV

## Goal

If the original MKV has working embedded subtitles, extract them and use them with the converted MP4.

This is better than using a random external `.srt`, because the embedded subtitles are already synced to the movie.

---

## Step 1 — Check Subtitle Streams in the Original MKV

Run:

```powershell
ffprobe -v error -show_entries stream=index,codec_type,codec_name:stream_tags=language,title -of default=noprint_wrappers=1 "input.mkv"
```

Look for a subtitle stream like this:

```text
index=2
codec_name=subrip
codec_type=subtitle
TAG:language=eng
```

In this example, the English subtitle stream is:

```text
index=2
```

---

## Step 2 — Extract the Embedded Subtitle

If the subtitle index is `2`, run:

```powershell
ffmpeg -i "input.mkv" -map 0:2 "movie-fixed.srt"
```

If the subtitle index is different, adjust the number.

Example for index `3`:

```powershell
ffmpeg -i "input.mkv" -map 0:3 "movie-fixed.srt"
```

---

## Step 3 — Match the File Names

For external subtitles, the video and subtitle should have the same base name:

```text
movie-fixed.mp4
movie-fixed.srt
```

Copy both files to the USB flash drive.

---

## Step 4 — Test on PC First

Open `movie-fixed.mp4` in VLC.

Check that the subtitle loads and is synced.

If it works on PC, copy both files to the USB:

```text
movie-fixed.mp4
movie-fixed.srt
```

Then test on the Sony player.

---

# Optional — Embed Subtitles into MP4

Use this if you do not want a separate `.srt` file.

This does **not** re-encode the video, so there is no video quality loss.

```powershell
ffmpeg -i "movie-fixed.mp4" -i "movie-fixed.srt" -c:v copy -c:a copy -c:s mov_text "movie-fixed-with-subs.mp4"
```

Then copy only this file to USB:

```text
movie-fixed-with-subs.mp4
```

Use this if the Sony handles embedded MP4 subtitles better than external `.srt` files.

---

# Optional — Burn Subtitles into the Video

Use this only if the Sony refuses external or embedded subtitles.

This permanently adds the subtitles into the picture.

```powershell
ffmpeg -i "movie-fixed.mp4" -vf "subtitles=movie-fixed.srt" -c:v libx264 -preset medium -crf 18 -profile:v high -level:v 4.1 -pix_fmt yuv420p -c:a copy "movie-fixed-burned-subs.mp4"
```

Result:

```text
movie-fixed-burned-subs.mp4
```

Important notes:

```text
Subtitles cannot be turned off.
Video is re-encoded again.
Compatibility is usually best.
```

Only burn subtitles if they are already synced when tested on your PC.

---

# Quick Reusable Workflow

For each movie, first check codecs and subtitle streams:

```powershell
ffprobe -v error -show_entries stream=index,codec_type,codec_name:stream_tags=language,title -of default=noprint_wrappers=1 "input.mkv"
```

Convert to Sony-compatible MP4:

```powershell
ffmpeg -i "input.mkv" -c:v libx264 -preset medium -crf 18 -profile:v high -level:v 4.1 -pix_fmt yuv420p -c:a aac -b:a 256k -movflags +faststart "movie-fixed.mp4"
```

Extract embedded English subtitle if available:

```powershell
ffmpeg -i "input.mkv" -map 0:2 "movie-fixed.srt"
```

Use matching names:

```text
movie-fixed.mp4
movie-fixed.srt
```

Copy both files to the USB flash drive and test them on the Sony player.

---

# Notes

## When Video Plays but There Is No Picture

If the MP4 only plays audio, the video codec is probably still unsupported, for example AV1 or HEVC.

Check with:

```powershell
ffprobe -v error -show_entries stream=index,codec_type,codec_name -of default=noprint_wrappers=1 "movie-fixed.mp4"
```

You want:

```text
Video: h264
Audio: aac
```

## When Subtitles Are Out of Sync

Do not burn subtitles immediately.

First test the converted MP4 and SRT on PC in VLC.

If the subtitles are out of sync on PC, the subtitle file is probably wrong for that release. Extract the embedded subtitle from the original MKV if available.

## When Sony Says “Operation Prohibited” in Subtitle Menu

This usually means the Sony player does not recognize the subtitle track or subtitle file for that USB playback mode.

Try this order:

```text
1. External .srt with matching filename
2. Embedded MP4 subtitle using mov_text
3. Burn subtitles into the video
```