# ffmpeg cheatsheet

ffmpeg 6.x. Order matters: input flags go before `-i`, output flags after.
`-c copy` = remux without re-encoding (fast, lossless).

## Inspect

```bash
ffprobe -v error -show_format -show_streams input.mp4
ffprobe -v error -select_streams v:0 -show_entries stream=width,height,codec_name,r_frame_rate input.mp4
ffprobe -v error -show_entries format=duration -of csv=p=0 input.mp4
```

## Convert / remux

```bash
ffmpeg -i in.mp4 out.mkv                 # container change, re-encode
ffmpeg -i in.mkv -c copy out.mp4         # remux only, no re-encode (fast)
ffmpeg -i in.webm -c:v libx264 -c:a aac out.mp4
ffmpeg -i in.mp4 -c:v libx265 -crf 28 out.mp4    # h265, smaller
ffmpeg -i in.mov -vn out.mp3             # strip video
ffmpeg -i in.mp4 -an out.mp4             # strip audio
```

## Trim

```bash
# -ss before -i = fast seek (keyframe-accurate-ish)
ffmpeg -ss 00:01:30 -i in.mp4 -t 00:00:30 -c copy out.mp4

# accurate cut (re-encodes, slow but frame-accurate)
ffmpeg -i in.mp4 -ss 00:01:30 -to 00:02:00 -c:v libx264 -c:a aac out.mp4

# from 10s to end
ffmpeg -ss 10 -i in.mp4 -c copy out.mp4
```

`-t` is a duration, `-to` is an end timestamp.

## Scale / crop

```bash
ffmpeg -i in.mp4 -vf scale=1280:-1 out.mp4       # width 1280, keep aspect
ffmpeg -i in.mp4 -vf scale=-2:720 out.mp4        # height 720 (even width)
ffmpeg -i in.mp4 -vf scale=640:480 out.mp4       # exact, may distort
ffmpeg -i in.mp4 -vf "crop=640:480:100:50" out.mp4   # w:h:x:y
ffmpeg -i in.mp4 -vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2" out.mp4
```

Use even dimensions for h264 (`-2` instead of `-1` when the dimension must be
computed). Odd widths make libx264 complain.

## Compress / quality

```bash
# CRF: lower = better quality + bigger. 18-28 is the useful range.
ffmpeg -i in.mp4 -c:v libx264 -crf 23 -preset medium -c:a aac -b:a 128k out.mp4
ffmpeg -i in.mp4 -c:v libx264 -crf 28 -preset slow out.mp4    # smaller
ffmpeg -i in.mp4 -b:v 2M -maxrate 2.5M -bufsize 4M out.mp4    # target bitrate
ffmpeg -i in.mp4 -crf 26 -vf scale=1280:-2 out.mp4
```

`-preset` trades speed for size: ultrafast ... veryslow.

## Audio

```bash
ffmpeg -i in.mp4 -c:a aac -b:a 192k out.mp4
ffmpeg -i in.wav -c:a libmp3lame -q:a 2 out.mp3
ffmpeg -i in.mp4 -af "volume=2.0" louder.mp4      # 2x volume
ffmpeg -i in.mp4 -af "loudnorm" normalized.mp4    # EBU R128 loudness
ffmpeg -i in.mp4 -af "atempo=1.5" fast.mp4        # 1.5x speed, pitch kept
ffmpeg -i in.mp4 -af "areverse" reversed.mp4
ffmpeg -i in.mp4 -vn -ac 2 -ar 44100 out.wav
ffmpeg -i a.mp3 -i b.mp3 -filter_complex "[0:a][1:a]concat=n=2:v=0:a=1" out.mp3
```

## Video speed / effects

```bash
ffmpeg -i in.mp4 -vf "setpts=0.5*PTS" -an fast.mp4    # 2x speed, no audio
ffmpeg -i in.mp4 -vf "setpts=2.0*PTS" slow.mp4        # 0.5x speed
ffmpeg -i in.mp4 -vf "fps=10" out.gif                 # gif (big; see below)
ffmpeg -i in.mp4 -vf "transpose=1" rotated.mp4        # 90 deg clockwise
ffmpeg -i in.mp4 -vf "hflip" mirrored.mp4
ffmpeg -i in.mp4 -vf "eq=brightness=0.1:contrast=1.2" tuned.mp4
```

## GIF (do it in two passes for decent quality)

```bash
ffmpeg -i in.mp4 -vf "fps=12,scale=480:-1:flags=lanczos,palettegen" palette.png
ffmpeg -i in.mp4 -i palette.png -lavfi "fps=12,scale=480:-1:flags=lanczos[x];[x][1:v]paletteuse" out.gif
```

## Thumbnails / frames

```bash
ffmpeg -i in.mp4 -ss 5 -frames:v 1 thumb.png          # one frame at 5s
ffmpeg -i in.mp4 -vf "fps=1/10" frame_%03d.png        # one every 10s
ffmpeg -i in.mp4 -vf "select='eq(n,100)'" -vframes 1 f.png
ffmpeg -i in.mp4 -vf "fps=1,scale=320:-1,tile=5x4" contact_sheet.png   # grid
```

## Concatenate

```bash
# same codec/params: use the concat demuxer (fast)
printf "file '%s'\n" a.mp4 b.mp4 c.mp4 > list.txt
ffmpeg -f concat -safe 0 -i list.txt -c copy out.mp4

# different sources: re-encode
ffmpeg -i a.mp4 -i b.mp4 -filter_complex "[0:v][0:a][1:v][1:a]concat=n=2:v=1:a=1" out.mp4
```

## Subtitles

```bash
ffmpeg -i in.mkv -map 0:s:0 out.srt                   # extract subs
ffmpeg -i in.mp4 -i subs.srt -c copy -c:s mov_text out.mp4   # mux into mp4
ffmpeg -i in.mkv -c copy -map 0 out.mkv               # copy all streams
ffmpeg -i in.mp4 -vf "subtitles=subs.srt" burned.mp4  # burn in
```

## Streams / mapping

```bash
ffmpeg -i in.mkv -map 0:v:0 -map 0:a:1 -c copy out.mkv  # first video + 2nd audio
ffmpeg -i in.mp4 -map 0 -c copy out.mkv                 # all streams
ffmpeg -i in.mp4 -hide_banner -loglevel error out.mp4   # quiet output
```

## Notes

- `-c copy` is the trick to remember: remux instead of re-encode. Wrong container
  with the right codecs costs seconds, not minutes.
- Put `-ss` before `-i` for a fast, keyframe-approximate seek. Put it after for
  accuracy at the cost of decoding from the start.
- CRF is quality-based; bitrate flags (`-b:v`) are size-based. Pick one.
- `ffprobe` before you guess. Half of "ffmpeg is broken" is a codec the container
  can't hold.
