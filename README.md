# MP4 Video Merger & Audio Processor

A comprehensive bash script that merges multiple MP4 files, splits the result into any number of cut segments, automatically normalizes audio to -0.5dB peak, and extracts high-quality MP3 audio files — with full execution logging.

Intended to be used with the output fragmented videos from OBS

## 🎬 Features

- **Smart MP4 Merging** - Combines multiple MP4 files while preserving original format and quality
- **Multi-Point Video Splitting** - Split the merged video into any number of segments by entering consecutive cut points (not just a single trim)
- **Intelligent Audio Normalization** - Automatically normalizes audio to -0.5dB peak (skips if already optimal), applied to the merged video or to each cut segment
- **High-Quality MP3 Extraction** - Creates 320kbps, 48kHz MP3 files automatically for the merged video or for each cut segment
- **Format Preservation** - No re-encoding during merge and cut operations for maximum quality
- **Comprehensive Video Analysis** - Shows codec information, resolution, duration, and audio specs
- **Smart File Management** - Handles existing files with skip/override options
- **Consistent Output Naming** - All files use customizable prefix for easy organization
- **Execution Logging** - Every run writes a full timestamped log (screen + file) to `logs/`, including the exact line/command on failure

## 📋 Requirements

- **FFmpeg** - Must be installed and available in PATH
- **Bash** - Ubuntu/Linux, or Windows via **Git Bash** (the script auto-detects `cygpath` and converts paths so FFmpeg, a native Windows binary, can read them correctly)
- **Dependencies** - At least one of: `bc`, `python3`, or `awk` (for floating-point calculations)

### Installing FFmpeg

```bash
# Ubuntu/Debian
sudo apt update && sudo apt install ffmpeg

# Check installation
ffmpeg -version
```

On Windows, install FFmpeg and make sure `ffmpeg.exe` is on your `PATH`, then run the script from Git Bash.

## 🚀 Installation

1. Download the script:
```bash
wget https://your-repo/merge_mp4_files.sh
# or
curl -O https://your-repo/merge_mp4_files.sh
```

2. Make it executable:
```bash
chmod +x merge_mp4_files.sh
```

## 📖 Usage

```bash
./merge_mp4_files.sh [input_directory] [output_prefix]
```

### Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `input_directory` | Directory containing MP4 files | Current directory (`.`) |
| `output_prefix` | Prefix for all output files | `processed` |

### Examples

```bash
# Basic usage - processes current directory
./merge_mp4_files.sh
# Output: processed_merged.mp4, plus normalized/mp3 or cut segments depending on your answers

# Custom directory
./merge_mp4_files.sh ./my_videos
# Output: processed_* files from ./my_videos

# Custom prefix
./merge_mp4_files.sh ./videos project
# Output: project_merged.mp4, project_normalized.mp4, project_audio.mp3, ...
```

## 📁 Output Files

The merged video is always created. What comes after depends on how many **cut points** you enter:

| File | Description | When Created |
|------|-------------|--------------|
| `{prefix}_merged.mp4` | Merged video from all MP4s | ✅ Always |
| `{prefix}_normalized.mp4` | Audio normalized to -0.5dB | Only if you enter **0** cut points |
| `{prefix}_audio.mp3` | High-quality MP3 (320kbps) | Only if you enter **0** cut points |
| `{prefix}_cut_N.mp4` | Segment N of the split video | One per segment, if you enter 1+ cut points |
| `{prefix}_cut_N_normalized.mp4` | Segment N, audio normalized to -0.5dB | One per segment created |
| `{prefix}_cut_N_audio.mp3` | Segment N, high-quality MP3 | One per segment created |
| `logs/merge_<timestamp>.log` | Full log of the run (screen output + errors) | ✅ Always |

## 🔄 Workflow

### 1. **Video Analysis & Merging**
- Scans directory for MP4 files (sorted alphabetically)
- Analyzes first file for format information (codec, resolution, framerate)
- Checks if merged file exists (skip/override/cancel options)
- Merges files using FFmpeg concat demuxer (no re-encoding)

### 2. **Optional Multi-Point Video Splitting**
- Asks how many **cut points** you want (`0` = no splitting)
- For each cut point, asks for a timestamp greater than the previous one and within the total duration
- Builds consecutive segments automatically: `00:00:00 → point 1 → point 2 → ... → end of video`
- If the last cut point you enter equals the total duration, no trailing segment is created (nothing left to cut)
- Each segment is cut independently from the original merged file (segments are not chained from one another)
- Flexible timestamp formats: `HH:MM:SS`, `MM:SS`, or `SS`
- Preserves format during cutting (stream copy)

### 3. **Automatic Audio Processing**
Runs once per output video — the merged file if you chose 0 cut points, or once per resulting segment otherwise:
- **Smart Normalization**: Analyzes current peak levels
  - If ≥ -0.5dB: Renames file (no processing needed)
  - If < -0.5dB: Applies calculated gain to reach -0.5dB
- **MP3 Extraction**: Creates high-quality MP3 from the normalized video

### 4. **Results Summary**
- Shows all created files with sizes and durations
- Displays a final processing summary with checkmarks for every file created (merged video, and every segment + its normalized video + its MP3)

## 💡 Example Session

```bash
$ ./merge_mp4_files.sh ./vacation_clips final

Log de esta ejecucion: /path/to/vacation_clips/logs/merge_20260819_101500.log
[INFO] MP4 File Merger, Cutter & Audio Processor
[INFO] Output prefix: final

[INFO] Found 5 MP4 files
[INFO] Video codec: h264, Audio codec: aac
[INFO] Resolution: 1920x1080, Frame rate: 30/1

[SUCCESS] Files merged successfully! (final_merged.mp4)
[DURATION] Total merged video duration: 02:15:30 (8130 seconds)

How many cut points do you want? (0 for none): 2
[INFO] Total video duration: 02:15:30
[INFO] Enter 2 cut point(s) in increasing order.
[INFO] The video will be split as: 00:00:00 -> point 1 -> point 2 -> ... -> end of video
   cut point 1: 00:45:00
   cut point 2: 01:30:00

--- Segment 1 of 3: 00:00:00 -> 00:45:00 ---
[SUCCESS] Segment 1 completed!
[AUDIO] Audio Processing Pipeline
[SUCCESS] Audio normalization completed!
[SUCCESS] MP3 extraction completed!

--- Segment 2 of 3: 00:45:00 -> 01:30:00 ---
[SUCCESS] Segment 2 completed!
...

--- Segment 3 of 3: 01:30:00 -> 02:15:30 ---
[SUCCESS] Segment 3 completed!
...

[SUCCESS] Processing Summary
✓ Merged video: final_merged.mp4
✓ Cut 1 video (00:00:00 -> 00:45:00): final_cut_1.mp4
✓ Cut 1 normalized: final_cut_1_normalized.mp4
✓ Cut 1 MP3 audio: final_cut_1_audio.mp3
✓ Cut 2 video (00:45:00 -> 01:30:00): final_cut_2.mp4
✓ Cut 2 normalized: final_cut_2_normalized.mp4
✓ Cut 2 MP3 audio: final_cut_2_audio.mp3
✓ Cut 3 video (01:30:00 -> 02:15:30): final_cut_3.mp4
✓ Cut 3 normalized: final_cut_3_normalized.mp4
✓ Cut 3 MP3 audio: final_cut_3_audio.mp3
```

## 🎯 Timestamp Format Examples

| Input | Interpretation | Seconds |
|-------|----------------|---------|
| `30` | 30 seconds | 30 |
| `5:30` | 5 minutes 30 seconds | 330 |
| `1:05:30` | 1 hour 5 minutes 30 seconds | 3930 |
| `00` | 0 seconds (start) | 0 |
| `90` | 90 seconds (1.5 minutes) | 90 |

## 🔧 Advanced Features

### Existing File Handling
When a merged file already exists:
- **Skip (s)**: Use existing file, proceed to cutting/audio
- **Override (o)**: Delete existing, create fresh merge
- **Cancel (c)**: Exit script

### Audio Normalization Intelligence
- Analyzes current peak levels before processing
- Skips normalization if audio is already ≥ -0.5dB
- Moves/renames file instead of copying when skipping
- Provides clear feedback about what was done

### Format Preservation
- Uses FFmpeg stream copy (`-c copy`) whenever possible
- No quality loss during merge and cut operations
- Maintains original codecs, resolution, and framerate

### Execution Logging
- Every run creates `logs/merge_<YYYYMMDD>_<HHMMSS>.log` next to the script
- All screen output (info, success, warnings, errors) is duplicated into the log via `tee`
- If the script fails, a trap reports the exact line number and command that caused it, both on screen and in the log — no more guessing what went wrong from a closed terminal

## 🛠️ Troubleshooting

### Common Issues

**Error: "FFmpeg not found"**
```bash
# Install FFmpeg
sudo apt update && sudo apt install ffmpeg
```

**Error: "No MP4 files found"**
- Check directory path
- Ensure files have `.mp4` extension (case sensitive)
- Verify read permissions

**Error: "Failed to merge files"**
- MP4 files may have incompatible formats
- Try re-encoding: `ffmpeg -i input.mp4 -c:v libx264 -c:a aac output.mp4`

**Error: "Impossible to open 'C:/c/...'" (Windows / Git Bash only)**
- This means paths weren't converted to Windows format before being handed to FFmpeg (a native Windows binary that doesn't understand Git Bash's `/c/...` POSIX paths). The script already converts these automatically via `cygpath -m` — make sure `cygpath` is available (it ships with Git Bash by default).

**Start time / cut point rejected even though it matches the shown duration**
- Cut points must be strictly less than the total duration — a point equal to the end of the video leaves nothing to cut. If you want the last segment to reach the very end, either omit that final cut point (the trailing segment is added automatically) or use a value slightly before the total duration.

**Error: "Numerical result out of range"**
- Usually resolved in latest version
- Ensure `bc`, `python3`, or `awk` is installed

### Diagnosing failures
Check `logs/merge_<timestamp>.log` for the run that failed — it contains the full output plus the `[ERROR] Fallo en la linea N ejecutando: <comando>` line pointing at the exact command that failed.

### Performance Tips

- **SSD Storage**: Process files on SSD for faster operations
- **File Organization**: Keep source files in same directory
- **Batch Processing**: Use consistent prefixes for easy file management

## 📊 Technical Specifications

### Video Processing
- **Merge Method**: FFmpeg concat demuxer
- **Quality**: Lossless (stream copy)
- **Supported Formats**: Any MP4-compatible codecs
- **Performance**: Limited by disk I/O, not CPU

### Audio Processing
- **Normalization Target**: -0.5dB peak
- **Analysis Method**: FFmpeg volumedetect filter
- **MP3 Quality**: 320kbps, 48kHz sampling rate
- **Codec**: libmp3lame

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🤝 Contributing

Contributions welcome! Please test thoroughly before submitting pull requests.

## 📞 Support

For issues or questions:
1. Check the troubleshooting section
2. Check `logs/merge_<timestamp>.log` for the failing run
3. Verify FFmpeg installation
4. Test with small sample files first
5. Check file permissions and paths

---

**Happy Video Processing! 🎬🎵**
