# FFmpeg Stats Linemode

Display multiple media file statistics inside the linemode column using `ffprobe` (part of FFmpeg). This plugin provides 9 independent toggleable stats:

- **Duration** - Media length (HH:MM:SS format)
- **Resolution** - Video dimensions (e.g., 1920x1080)
- **Codec** - Video codec (e.g., H264, HEVC)
- **FPS** - Frame rate (e.g., 30fps, 29.97fps)
- **Bitrate** - Overall bitrate (e.g., 5.2Mbps, 320kbps)
- **Audio Codec** - Audio codec (e.g., AAC, OPUS)
- **Audio Channels** - Channel configuration (e.g., stereo, 5.1ch)
- **Format** - Container format (e.g., MP4, MKV)
- **Aspect Ratio** - Display aspect ratio (e.g., 16:9, 21:9)

All stats are cached per file and fetched efficiently in a single `ffprobe` call. Each stat can be toggled independently and sorted with automatic linemode restoration.

## Requirements

- `ffprobe` available in your `PATH` (`ffprobe` ships with FFmpeg)
- Yazi build that supports sorting by linemode

## Installation

1. Ensure the plugin files live at `~/.config/yazi/plugins/ffmpeg-stats-lm.yazi/`

2. Register the fetcher so Yazi runs `ffprobe` in the background:

   ```toml
   # In ~/.config/yazi/yazi.toml
   [[plugin.prepend_fetchers]]
   id  = "ffmpeg_stats_lm"
   url = "*"  # For Yazi source build; use `name = "*"` for v0.4.0 and earlier
   run = "ffmpeg-stats-lm"
   ```

3. Load the plugin in `~/.config/yazi/init.lua`:

   ```lua
   require("ffmpeg-stats-lm"):setup({
       -- Enable specific stats by default (all start disabled)
       duration = false,
       resolution = true,   -- Example: enable resolution by default
       codec = true,        -- Example: enable codec by default
       fps = false,
       bitrate = false,
       audio_codec = false,
       audio_channels = false,
       format = false,
       aspect = false,

       -- Optional: custom styling
       style = ui.Style():fg("cyan"),
       order = 1600,  -- Display order in linemode
   })
   ```

## Usage

### Toggle Commands

Each stat can be toggled independently:

```toml
# In ~/.config/yazi/keymap.toml

# Toggle individual stats
{ on = [ "m", "d" ], run = "plugin ffmpeg-stats-lm -- toggle-duration", desc = "Toggle duration" },
{ on = [ "m", "r" ], run = "plugin ffmpeg-stats-lm -- toggle-resolution", desc = "Toggle resolution" },
{ on = [ "m", "c" ], run = "plugin ffmpeg-stats-lm -- toggle-codec", desc = "Toggle codec" },
{ on = [ "m", "f" ], run = "plugin ffmpeg-stats-lm -- toggle-fps", desc = "Toggle FPS" },
{ on = [ "m", "b" ], run = "plugin ffmpeg-stats-lm -- toggle-bitrate", desc = "Toggle bitrate" },
{ on = [ "m", "a" ], run = "plugin ffmpeg-stats-lm -- toggle-audio-codec", desc = "Toggle audio codec" },
{ on = [ "m", "h" ], run = "plugin ffmpeg-stats-lm -- toggle-audio-channels", desc = "Toggle audio channels" },
{ on = [ "m", "o" ], run = "plugin ffmpeg-stats-lm -- toggle-format", desc = "Toggle format" },
{ on = [ "m", "s" ], run = "plugin ffmpeg-stats-lm -- toggle-aspect", desc = "Toggle aspect ratio" },

# Bulk toggle operations
{ on = [ "m", "A" ], run = "plugin ffmpeg-stats-lm -- toggle-all", desc = "Toggle all stats" },
{ on = [ "m", "D" ], run = "plugin ffmpeg-stats-lm -- disable-all", desc = "Disable all stats" },
```

### Sort Commands

Sort by any stat while preserving your current linemode:

```toml
# Sort commands
{ on = [ ",", "d" ], run = "plugin ffmpeg-stats-lm -- sort-duration", desc = "Sort by duration" },
{ on = [ ",", "D" ], run = "plugin ffmpeg-stats-lm -- sort-duration-reverse", desc = "Sort by duration (reverse)" },
{ on = [ ",", "r" ], run = "plugin ffmpeg-stats-lm -- sort-resolution", desc = "Sort by resolution" },
{ on = [ ",", "R" ], run = "plugin ffmpeg-stats-lm -- sort-resolution-reverse", desc = "Sort by resolution (reverse)" },
{ on = [ ",", "c" ], run = "plugin ffmpeg-stats-lm -- sort-codec", desc = "Sort by codec" },
{ on = [ ",", "f" ], run = "plugin ffmpeg-stats-lm -- sort-fps", desc = "Sort by FPS" },
{ on = [ ",", "b" ], run = "plugin ffmpeg-stats-lm -- sort-bitrate", desc = "Sort by bitrate" },
```

Available sort commands (internal linemode names in parentheses):
- `sort-duration` / `sort-duration-reverse` (uses `ffmpeg_duration_sort`)
- `sort-resolution` / `sort-resolution-reverse` (uses `ffmpeg_res_sort`)
- `sort-codec` / `sort-codec-reverse` (uses `ffmpeg_codec_sort`)
- `sort-fps` / `sort-fps-reverse` (uses `ffmpeg_fps_sort`)
- `sort-bitrate` / `sort-bitrate-reverse` (uses `ffmpeg_bitrate_sort`)
- `sort-audio-codec` / `sort-audio-codec-reverse` (uses `ffmpeg_acodec_sort`)
- `sort-audio-channels` / `sort-audio-channels-reverse` (uses `ffmpeg_channels_sort`)
- `sort-format` / `sort-format-reverse` (uses `ffmpeg_format_sort`)
- `sort-aspect` / `sort-aspect-reverse` (uses `ffmpeg_aspect_sort`)

### Direct Linemode Usage

You can also use individual stats as direct linemodes:

```lua
-- In Yazi configuration
linemode = "ffmpeg_resolution"  -- Show only resolution
linemode = "ffmpeg_codec"       -- Show only codec
linemode = "ffmpeg_duration"    -- Show only duration
-- etc.
```

Available linemodes:
- `ffmpeg_duration` - Display: "01:23:45"
- `ffmpeg_resolution` - Display: "1920x1080"
- `ffmpeg_codec` - Display: "H264"
- `ffmpeg_fps` - Display: "30fps"
- `ffmpeg_bitrate` - Display: "5.2Mbps"
- `ffmpeg_audio_codec` - Display: "AAC"
- `ffmpeg_audio_channels` - Display: "stereo"
- `ffmpeg_format` - Display: "MP4"
- `ffmpeg_aspect` - Display: "16:9"

For sorting, use the corresponding `*_sort` linemode (e.g., `ffmpeg_duration_sort`, `ffmpeg_res_sort`), which uses the same formatting for readability.

**Note**: Yazi requires linemode names to be 20 characters or less, so some sort linemodes use abbreviated names (e.g., `ffmpeg_res_sort` instead of `ffmpeg_resolution_sort`).

## How It Works

1. **Background Fetching**: When you enter a directory with media files, the plugin automatically runs `ffprobe` in the background via Yazi's fetcher system.

2. **Efficient Caching**: All 9 stats are fetched with a single `ffprobe` call per file (~100ms) and cached. Subsequent toggles are instant.

3. **Conditional Display**: Only enabled stats appear in the linemode column. Toggle any combination you want.

4. **Smart Sorting**: Sort commands temporarily switch to a readable sort linemode (using the same formatting as display), perform the sort, then automatically restore your previous linemode.

## Usage Notes

- Toggle stats on/off as needed - only enabled stats consume visual space
- Sorting works even if the stat isn't currently visible
- Audio-only files (MP3, FLAC, etc.) will only show audio stats and duration
- Video-only files will only show video stats
- Stats are cached per-file until Yazi restarts
- Set `YAZI_FFMPEG_STATS_DEBUG=1` before launching Yazi to inspect debug logs in `~/.local/state/yazi/yazi.log`

## Examples

### Minimal Setup (Resolution + Codec only)

```lua
require("ffmpeg-stats-lm"):setup({
    resolution = true,
    codec = true,
})
```

### Media Library Setup (All Video Stats)

```lua
require("ffmpeg-stats-lm"):setup({
    duration = true,
    resolution = true,
    codec = true,
    fps = true,
    bitrate = true,
    aspect = true,
})
```

### Audio-Focused Setup

```lua
require("ffmpeg-stats-lm"):setup({
    duration = true,
    bitrate = true,
    audio_codec = true,
    audio_channels = true,
    format = true,
})
```

## Troubleshooting

### No stats appearing

1. Ensure `ffprobe` is installed: `which ffprobe`
2. Check the fetcher is registered in `yazi.toml`
3. Check the plugin is loaded in `init.lua`
4. Try toggling a stat with your keybinding
5. Enable debug mode and check logs:
   ```bash
   YAZI_FFMPEG_STATS_DEBUG=1 yazi
   # Then check: ~/.local/state/yazi/yazi.log
   ```

### Stats not updating

- Stats are cached per-file. Restart Yazi to refresh the cache, or use the `disable-all` command to clear cached data.

### Fetcher registration for different Yazi versions

- **Yazi v0.4.0 and earlier**: Use `name = "*"` in the fetcher registration
- **Yazi source build (main branch)**: Use `url = "*"` in the fetcher registration

## Comparison with ffmpeg-duration-lm

This plugin is inspired by and compatible with `ffmpeg-duration-lm`. Key differences:

- **Multiple stats**: 9 independent stats vs. duration only
- **Same format**: Duration uses the same HH:MM:SS format
- **Same patterns**: Toggle system, sort system, and fetcher pattern all follow the same design
- **Separate keybindings**: Each stat has its own toggle and sort commands
- **Compatible**: Can be used alongside `ffmpeg-duration-lm` if desired

## License

Same as Yazi.
