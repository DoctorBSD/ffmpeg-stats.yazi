# ffmpeg-stats.yazi

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

All stats are cached per file and fetched efficiently in a single `ffprobe` call.

## Version Compatibility

This plugin supports two configurations depending on your Yazi version:

**Current Yazi (official release)**
- Linemode display only (toggle stats on/off)
- See [Current Version Usage](#current-version-usage) below

**Future Yazi (custom fork with sorting-by-linemode support)**
- Full linemode display + sorting support
- Requires my fork
- If my PR is merged then this functionality will be available in the official version of yazi
- See [Future Version Usage (with Sorting)](#future-version-usage-with-sorting) below

## Requirements

- `ffprobe` available in your `PATH` (`ffprobe` ships with FFmpeg)

---

# Current Version Usage

For Yazi v0.4.0 and earlier, or main branch builds without sorting support.

## Installation

1. Clone this plugin

    ```
    git clone https://github.com/grimandgreedy/ffmpeg-stats.yazi ~/.config/yazi/plugins
    ```

2. Register the fetcher so Yazi can run `ffprobe` in the background:

```toml
# In ~/.config/yazi/yazi.toml
[[plugin.prepend_fetchers]]
id  = "ffmpeg_stats"
name = "*"
run = "ffmpeg-stats"
```

3. Load the plugin in `~/.config/yazi/init.lua`:

   ```lua
    require("ffmpeg-stats"):setup({
        -- Which stats should be shown by default upon opening yazi
        duration = false,
        resolution = false,
        codec = false,
        fps = false,
        bitrate = false,
        audio_codec = false,
        audio_channels = false,
        format = false,
        aspect = false,

        -- Uses theme colour by default
        -- style = ui.Style():fg("cyan"),
   })
   ```

## Keybindings

Add toggle commands to `~/.config/yazi/keymap.toml`:

```toml
# In ~/.config/yazi/keymap.toml

## ffmpeg linemodes - toggle individual stats
{ on = [ "m", "f", "d" ], run = "plugin ffmpeg-stats -- toggle-duration", desc = "Toggle duration" },
{ on = [ "m", "f", "r" ], run = "plugin ffmpeg-stats -- toggle-resolution", desc = "Toggle resolution" },
{ on = [ "m", "f", "c" ], run = "plugin ffmpeg-stats -- toggle-codec", desc = "Toggle codec" },
{ on = [ "m", "f", "f" ], run = "plugin ffmpeg-stats -- toggle-fps", desc = "Toggle FPS" },
{ on = [ "m", "f", "b" ], run = "plugin ffmpeg-stats -- toggle-bitrate", desc = "Toggle bitrate" },
{ on = [ "m", "f", "a" ], run = "plugin ffmpeg-stats -- toggle-audio-codec", desc = "Toggle audio codec" },
{ on = [ "m", "f", "h" ], run = "plugin ffmpeg-stats -- toggle-audio-channels", desc = "Toggle audio channels" },
{ on = [ "m", "f", "o" ], run = "plugin ffmpeg-stats -- toggle-format", desc = "Toggle format" },
{ on = [ "m", "f", "s" ], run = "plugin ffmpeg-stats -- toggle-aspect", desc = "Toggle aspect ratio" },

# Bulk toggle operations
{ on = [ "m", "f", "A" ], run = "plugin ffmpeg-stats -- toggle-all", desc = "Toggle all stats" },
{ on = [ "m", "f", "D" ], run = "plugin ffmpeg-stats -- disable-all", desc = "Disable all stats" },
```

## Direct Linemode Usage

You can use individual stats as direct linemodes in your settings:

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

- Stats are cached per-file. Restart Yazi to refresh the cache.

---

# Future Version Usage (with Sorting)

I created a fork of yazi in which I implemented the ability to sort by linemode values. This allows us to sort files by, say, duration. 

The functionality is implemented and working but I still need to find time to check it over before I make a PR. Once I do, hopefully, this will be available in the main version of yazi.


## Installation

1. Clone this plugin

    ```
    git clone https://github.com/grimandgreedy/ffmpeg-stats.yazi ~/.config/yazi/plugins
    ```

2. Register the fetcher so Yazi runs `ffprobe` in the background:

   ```toml
   # In ~/.config/yazi/yazi.toml
   [[plugin.prepend_fetchers]]
   id  = "ffmpeg_stats"
   url = "*"   # Note that url (not name) is needed for the new version of yazi
   run = "ffmpeg-stats"
   ```

3. Load the plugin in `~/.config/yazi/init.lua`:

   ```lua
    require("ffmpeg-stats"):setup({
        -- Which stats should be shown by default upon opening yazi
        duration = false,
        resolution = false,
        codec = false,
        fps = false,
        bitrate = false,
        audio_codec = false,
        audio_channels = false,
        format = false,
        aspect = false,

        -- Uses theme colour by default
        -- style = ui.Style():fg("cyan"),
   })
   ```

## Keybindings

Add both toggle and sort commands to `~/.config/yazi/keymap.toml`:

```toml
# In ~/.config/yazi/keymap.toml

## ffmpeg linemodes - toggle individual stats
{ on = [ "m", "f", "d" ], run = "plugin ffmpeg-stats -- toggle-duration", desc = "Toggle duration" },
{ on = [ "m", "f", "r" ], run = "plugin ffmpeg-stats -- toggle-resolution", desc = "Toggle resolution" },
{ on = [ "m", "f", "c" ], run = "plugin ffmpeg-stats -- toggle-codec", desc = "Toggle codec" },
{ on = [ "m", "f", "f" ], run = "plugin ffmpeg-stats -- toggle-fps", desc = "Toggle FPS" },
{ on = [ "m", "f", "b" ], run = "plugin ffmpeg-stats -- toggle-bitrate", desc = "Toggle bitrate" },
{ on = [ "m", "f", "a" ], run = "plugin ffmpeg-stats -- toggle-audio-codec", desc = "Toggle audio codec" },
{ on = [ "m", "f", "h" ], run = "plugin ffmpeg-stats -- toggle-audio-channels", desc = "Toggle audio channels" },
{ on = [ "m", "f", "o" ], run = "plugin ffmpeg-stats -- toggle-format", desc = "Toggle format" },
{ on = [ "m", "f", "s" ], run = "plugin ffmpeg-stats -- toggle-aspect", desc = "Toggle aspect ratio" },

# Bulk toggle operations
{ on = [ "m", "f", "A" ], run = "plugin ffmpeg-stats -- toggle-all", desc = "Toggle all stats" },
{ on = [ "m", "f", "D" ], run = "plugin ffmpeg-stats -- disable-all", desc = "Disable all stats" },

## ffmpeg sorting - sort by media stats
{ on = [ ",", "f", "d" ], run = "plugin ffmpeg-stats -- sort-duration", desc = "Sort by duration" },
{ on = [ ",", "f", "D" ], run = "plugin ffmpeg-stats -- sort-duration-reverse", desc = "Sort by duration (reverse)" },
{ on = [ ",", "f", "r" ], run = "plugin ffmpeg-stats -- sort-resolution", desc = "Sort by resolution" },
{ on = [ ",", "f", "R" ], run = "plugin ffmpeg-stats -- sort-resolution-reverse", desc = "Sort by resolution (reverse)" },
{ on = [ ",", "f", "c" ], run = "plugin ffmpeg-stats -- sort-codec", desc = "Sort by codec" },
{ on = [ ",", "f", "C" ], run = "plugin ffmpeg-stats -- sort-codec-reverse", desc = "Sort by codec (reverse)" },
{ on = [ ",", "f", "f" ], run = "plugin ffmpeg-stats -- sort-fps", desc = "Sort by FPS" },
{ on = [ ",", "f", "F" ], run = "plugin ffmpeg-stats -- sort-fps-reverse", desc = "Sort by FPS (reverse)" },
{ on = [ ",", "f", "b" ], run = "plugin ffmpeg-stats -- sort-bitrate", desc = "Sort by bitrate" },
{ on = [ ",", "f", "B" ], run = "plugin ffmpeg-stats -- sort-bitrate-reverse", desc = "Sort by bitrate (reverse)" },
{ on = [ ",", "f", "a" ], run = "plugin ffmpeg-stats -- sort-audio-codec", desc = "Sort by audio codec" },
{ on = [ ",", "f", "A" ], run = "plugin ffmpeg-stats -- sort-audio-codec-reverse", desc = "Sort by audio codec (reverse)" },
{ on = [ ",", "f", "h" ], run = "plugin ffmpeg-stats -- sort-audio-channels", desc = "Sort by audio channels" },
{ on = [ ",", "f", "H" ], run = "plugin ffmpeg-stats -- sort-audio-channels-reverse", desc = "Sort by audio channels (reverse)" },
{ on = [ ",", "f", "o" ], run = "plugin ffmpeg-stats -- sort-format", desc = "Sort by format" },
{ on = [ ",", "f", "O" ], run = "plugin ffmpeg-stats -- sort-format-reverse", desc = "Sort by format (reverse)" },
{ on = [ ",", "f", "s" ], run = "plugin ffmpeg-stats -- sort-aspect", desc = "Sort by aspect ratio" },
{ on = [ ",", "f", "S" ], run = "plugin ffmpeg-stats -- sort-aspect-reverse", desc = "Sort by aspect ratio (reverse)" },
```

# Direct Linemode Usage

You can use individual stats as direct linemodes in your settings:

```lua
-- In Yazi configuration
linemode = "ffmpeg_resolution"  -- Show resolution
linemode = "ffmpeg_codec"       -- Show codec
linemode = "ffmpeg_duration"    -- Show duration
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

**Note**: Yazi requires linemode names to be 20 characters or less, so some sort linemodes use abbreviated names (e.g., `ffmpeg_res_sort` instead of `ffmpeg_resolution_sort`).

Available sort linemodes (internal names):
- `ffmpeg_duration_sort` / `*_reverse`
- `ffmpeg_res_sort` / `*_reverse` (for resolution)
- `ffmpeg_codec_sort` / `*_reverse`
- `ffmpeg_fps_sort` / `*_reverse`
- `ffmpeg_bitrate_sort` / `*_reverse`
- `ffmpeg_acodec_sort` / `*_reverse` (for audio codec)
- `ffmpeg_channels_sort` / `*_reverse` (for audio channels)
- `ffmpeg_format_sort` / `*_reverse`
- `ffmpeg_aspect_sort` / `*_reverse`
