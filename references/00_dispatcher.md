# Media Delogo: remove visible generator watermarks

Removes the visible Gemini or Flow mark (sparkle or corner mark) from AI-generated images and video, from any tool (flow-mcp, Gemini, Veo, Nano Banana). Do it for every generated file before delivery, without asking first. Skip only when the user says so for that run.

## Limits

- This removes the visible mark only. Invisible provenance marks (SynthID) are untouched. Never claim otherwise.
- Do not install torch, LaMa or other heavy dependencies without asking.
- Tools used: ffmpeg and ImageMagick only. `magick` is ImageMagick 7; on ImageMagick 6 use `convert`.

## Rules

- Keep the original untouched. Write the cleaned copy beside it as `<name>-clean.<ext>` and deliver that path.
- Clean images that will be reused as references too.
- Quote every path in shell commands. Never overwrite an existing `-clean` file: pick a new name or ask. Write scratch frames to a temp directory, not the working folder.
- Re-encoding drops EXIF and C2PA metadata. Do not claim provenance data is intact.
- Templates use `.mp4`/`.png`; keep the input's extension, and drop `-c:a copy` if the container cannot hold the audio codec.
- If a crop changes the framing, say so once.

## Procedure

1. Get the size: `ffprobe -v error -select_streams v:0 -show_entries stream=width,height -of csv=p=0 IN`
2. Locate the mark in the actual file. It sits near the bottom-right corner. Starting points only: about 45 px at about 75 px inset on 1376x768 stills; about 47 px at about 96 px inset on 1280x720 video. Extract one frame (video) or crop the corner (still) and look at it:
   - Video frame: `ffmpeg -n -ss 1 -i IN -frames:v 1 frame.png`
   - Corner zoom: `magick frame.png -gravity southeast -crop 200x200+0+0 +repage -scale 400% corner.png`
3. Compute the box. Inset is the gap from the image edge to the mark's outer edge: `x = W - inset - size`, `y = H - inset - size`, `w = h = size`. Add 4 to 6 px of margin on each side. The box must lie fully inside the frame (x >= 1, y >= 1, x+w <= W-1, y+h <= H-1) or delogo fails.
4. Run the command for the file type. A non-zero exit or an ffmpeg `outside of the frame` message means the box is wrong: fix it, do not deliver.
5. Look at the result (step 6) before delivering.

## Commands

Video, delogo (re-encodes video, copies audio):

`ffmpeg -n -i IN.mp4 -vf "delogo=x=X:y=Y:w=W:h=H" -c:a copy OUT-clean.mp4`

Still, delogo (ffmpeg):

`ffmpeg -n -i IN.png -vf "delogo=x=X:y=Y:w=W:h=H" -frames:v 1 OUT-clean.png`

Still, ImageMagick fallback (paste the patch just left of the box over it; no framing change; needs X >= W+4):

`magick IN.png \( +clone -crop WxH+$((X-W-4))+Y +repage \) -geometry +X+Y -composite OUT-clean.png`

Crop (last resort, changes framing; crops the bottom and right edges; use even values for video):

- Video: `ffmpeg -n -i IN.mp4 -vf "crop=iw-CROPR:ih-CROPB:0:0" -c:a copy OUT-clean.mp4`
- Still: `magick IN.png -gravity northwest -crop WIDTHxHEIGHT+0+0 +repage OUT-clean.png`

## Check the result

6. View the output: for video, extract a frame with step 2 at several timestamps (the mark can move) and zoom the corner as in step 2. For stills, zoom the corner the same way.
7. A visible patch or smear means retry with a different box (larger, shifted, or more margin), then the ImageMagick patch, then a crop. Repeat the check each time.
8. Deliver the `-clean` path and state that only the visible mark was removed.
