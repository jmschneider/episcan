# Episcan

Match TV episodes using audio transcription and episode subtitles (default) or descriptions.

## Features

- **Audio Transcription**: Uses OpenAI Whisper to transcribe video audio (loaded on-demand)
- **Time-Synchronized Comparison**: Compares matching time segments for fair subtitle-to-subtitle comparison
- **Smart Subtitle Track Selection**: Finds best matching video subtitle track using similarity analysis
- **Comprehensive Subtitle Caching**: OS-appropriate cache storage with automatic management
- **Enhanced Subtitle Provider Support**:
  - Local subtitle files (`--subtitles-dir`)
  - Embedded video subtitles (`--try-subtitles`)
  - Subliminal library (default) - Multiple providers with authentication support
  - External ID matching (TMDB/TVDB/IMDB) for better provider accuracy
- **Robust Subtitle Retry System**:
  - Configurable retry attempts with exponential backoff (default: 5 retries)
  - Smart failure handling (exit/prompt/continue after retries exhausted)
- **Intelligent Transcription Defaults**:
  - Subtitle comparison: 3 minutes starting at 1 minute (skips intros)
  - Description comparison: Full episode transcription
- **Universal Subtitle Support**: Handles SRT, WebVTT, ASS/SSA, MicroDVD, MPL2, TMP, and JSON formats via pysubs2
- **Advanced File Management**: Smart conflict resolution for renaming with file preservation
- **Optimal Episode Matching**: Uses SBERT embeddings and Hungarian algorithm for unique assignments
- **GPU Acceleration**: CUDA support for both Whisper and sentence transformers
- **Memory Efficient**: Conditional model loading saves resources when not needed
- **Progress Tracking**: ETA calculations and detailed processing feedback
- **Multiple APIs**: TMDB (preferred) and TVDB support with external ID enrichment

## Quick Start

```bash
# Default: Use subliminal for subtitle downloads with 5 retry attempts
uv run python main.py /path/to/videos

# Use local subtitle files
uv run python main.py /path/to/videos --subtitles-dir /path/to/subtitles

# Compare against episode descriptions instead
uv run python main.py /path/to/videos --use-descriptions

# Custom retry behavior: 3 retries, then continue anyway
uv run python main.py /path/to/videos --subtitle-retries 3 --on-subtitle-failure continue

# Clear cache and disable caching for fresh downloads
uv run python main.py /path/to/videos --clear-cache --no-cache
```

## Usage Examples

### Basic Usage

```bash
# Default behavior: subtitle comparison using subliminal
episcan /path/to/videos

# Use local subtitle files for comparison
episcan /path/to/videos --subtitles-dir /path/to/subtitles

# Compare against episode descriptions (full transcription)
episcan /path/to/videos --use-descriptions

# Force full episode transcription even for subtitle comparison
episcan /path/to/videos --max-duration 0
```

### Advanced Options

```bash
# Custom models, auto-rename, verbose output
episcan /path/to/videos \
    --whisper-model medium \
    --sbert-model sentence-transformers/all-MiniLM-L6-v2 \
    --rename auto \
    --verbose

# Try embedded subtitles first, fallback to Whisper
episcan /path/to/videos --try-subtitles

# Cache management for re-processing shows
episcan /path/to/videos --clear-cache  # Clear existing cache
episcan /path/to/videos --no-cache     # Disable caching entirely

# Adjust transcription timing
episcan /path/to/videos --max-duration 300 --start-offset 30
```

### Environment Variables

```bash
export TMDB_API_KEY="your_tmdb_key"             # Preferred
export TVDB_API_KEY="your_tvdb_key"             # Fallback

# Optional: Subtitle provider authentication (improves success rates)
export ADDIC7ED_USERNAME="your_username"        # Addic7ed account
export ADDIC7ED_PASSWORD="your_password"
export OPENSUBTITLES_USERNAME="your_username"   # OpenSubtitles account  
export OPENSUBTITLES_PASSWORD="your_password"
```

## How It Works

### Subtitle Comparison (Default)

1. **Check Cache**: Looks for previously downloaded subtitles in OS-appropriate cache directory
2. **Download Subtitles**: Uses subliminal with enhanced provider configurations and external ID matching
3. **Enhanced Matching**: Utilizes TMDB/TVDB/IMDB IDs for better provider accuracy
4. **Retry System**: Automatically retries failed downloads with exponential backoff (2s→4s→8s→16s→32s)
5. **Cache Storage**: Saves downloaded subtitles with metadata for future use
6. **Time-Synchronized Extraction**: For partial transcription, extracts matching time segments from episode subtitles
7. **Smart Track Selection**: When using `--try-subtitles`, finds video subtitle track with best similarity to episode content
8. **Fair Comparison**: Compares equivalent content (3min transcript vs 3min subtitle segment)
9. **Optimal Assignment**: Uses Hungarian algorithm to ensure unique episode matches

### Description Comparison

1. **Full Transcription**: Transcribes entire episodes for comprehensive comparison
2. **Metadata Matching**: Compares transcripts against episode descriptions from TMDB/TVDB
3. **Similarity Scoring**: Uses cosine similarity with sentence transformers

## Subtitle Sources

**Priority Order:**

1. **Local Files** (`--subtitles-dir`) - Custom subtitle directory
2. **Subliminal** (default) - Enhanced provider support with authentication:
   - Multiple provider strategies with intelligent fallback
   - External ID matching (TMDB/TVDB/IMDB) for better accuracy
   - Optional authentication for Addic7ed and OpenSubtitles (via environment variables)
   - Configurable retry system with exponential backoff (default: 5 attempts)
   - OS-appropriate cache storage (macOS: `~/Library/Caches/episcan/`)
   - Cache hits eliminate re-downloads for repeated processing

## Why Subliminal?

- **Enhanced Provider Support**: Multiple subtitle providers with authentication for higher success rates
- **External ID Matching**: Uses TMDB/TVDB/IMDB IDs for more accurate content matching
- **Robust Retry System**: Automatic retries with exponential backoff for temporary failures
- **Smart Caching**: Prevents re-downloads with persistent storage and metadata
- **Format Support**: Handles various subtitle formats automatically
- **Respectful**: Built-in rate limiting and intelligent provider rotation
- **Zero Configuration**: Works without API keys, but supports authentication for better results

## Command Line Options

```
positional arguments:
  video_dir                Directory containing video files (default: current directory)

options:
  --tvdb-api-key           TVDB API key (or use TVDB_API_KEY environment variable)
  --tmdb-api-key           TMDB API key (or use TMDB_API_KEY environment variable)
  --subtitles-dir          Directory containing subtitle files for episodes
  --use-descriptions       Use episode descriptions instead of subtitles (default: use subtitles)
  --force-tvdb             Force TVDB even if TMDB key available
  --whisper-model          Whisper model: tiny, base, small, medium, large (default: base)
  --sbert-model            Sentence transformer model (default: all-mpnet-base-v2)
  --max-duration           Transcription duration in seconds (default: 180 for subtitles, 0=full)
  --start-offset           Skip intro seconds (default: 60)
  --rename                 File renaming: none, prompt, auto (default: none)
  --try-subtitles          Try embedded subtitles first, fallback to Whisper
  --subtitle-retries       Number of retry attempts for missing subtitles (default: 5)
  --on-subtitle-failure    Action when subtitles still missing after retries: exit, prompt, continue (default: exit)
  --clear-cache            Clear subtitle cache before processing
  --no-cache               Disable subtitle caching (always download fresh)
  --verbose                Detailed processing information
```

## Supported Subtitle Formats

Thanks to **pysubs2** integration, episcan supports all major subtitle formats:

- SubRip (`.srt`)
- WebVTT (`.vtt`)
- Advanced SubStation Alpha (`.ass`, `.ssa`)
- MicroDVD (`.sub`)
- MPL2 (`.mpl`)
- TMP (`.tmp`)
- JSON subtitles

## Example Output

```
Using TMDB API
Found series: Breaking Bad (TMDB ID: 1396, Year: 2008)
  External IDs - IMDB: tt0903747, TVDB: 81189
Loading sentence transformer model (sentence-transformers/all-mpnet-base-v2) on cuda...
Found 8 video files
Detected: Breaking Bad Season 1
Fetching episode subtitles using subliminal...
  Attempting to get subtitles for 8 episodes (checking cache first)...
✓ All episodes have subtitles
Processing 8 video files...
  1/8: episode1.mkv ✓ (12.3s)
  2/8: episode2.mkv ✓ (11.8s)
  ...

Calculating optimal matches...

=== FINAL MATCHES ===
✓ episode1.mkv -> S01E01 - Pilot
  Similarity: 0.847

→ episode5.mkv -> S01E05 - Gray Matter
  Similarity: 0.723

=== FILE RENAMING ===
Planned renames:
  episode1.mkv → Breaking Bad - S01E01 - Pilot.mkv
  episode5.mkv → Breaking Bad - S01E05 - Gray Matter.mkv

Renamed 8/8 files successfully
```

## Performance Tips

- **Caching Benefits**: Second runs on same show are dramatically faster with subtitle cache hits
- **GPU Acceleration**: Use CUDA-compatible hardware for 5-10x speed improvement
- **Memory Optimization**: Use `--try-subtitles` to avoid Whisper loading when embedded subtitles are available
- **Cache Management**:
  - Cache persists between runs for faster re-processing
  - Use `--clear-cache` when switching show versions or subtitle preferences
  - Cache stored in OS-appropriate locations (Linux: `~/.cache/episcan/`, Windows: `%LOCALAPPDATA%\episcan\`)
- **Model Selection**:
  - `whisper-model base`: Good balance of speed/accuracy (loaded on-demand)
  - `sbert-model all-MiniLM-L6-v2`: Faster but less accurate than default
- **Transcription Optimization**:
  - Default 3-minute excerpts work well for most shows with time-synchronized comparison
  - Use `--max-duration 0` for very short episodes or poor matches
  - Adjust `--start-offset` for shows with long intros
- **Subtitle Extraction**: `--try-subtitles` can be much faster than transcription for videos with embedded subs

## Troubleshooting

### Common Issues

**Missing subtitles despite retries:**

```bash
# Set provider credentials for better access
export ADDIC7ED_USERNAME="your_user"
export ADDIC7ED_PASSWORD="your_pass"

# Increase retry attempts
episcan /path/to/videos --subtitle-retries 10

# Continue anyway with partial coverage
episcan /path/to/videos --on-subtitle-failure continue

# Use description comparison as fallback
episcan /path/to/videos --use-descriptions
```

**File renaming conflicts:**

```bash
# Conflicts are automatically resolved with UUID preservation
# Files are never deleted - conflicting files get UNMATCHED_ prefix
# Example: "Show - S01E01.mkv" becomes "UNMATCHED_Show - S01E01_a1b2c3d4.mkv"
```

**Poor matching accuracy:**

```bash
# Try full episode transcription
episcan /path/to/videos --max-duration 0

# Use higher quality models
episcan /path/to/videos --whisper-model medium --sbert-model sentence-transformers/all-mpnet-base-v2
```

**Subtitle provider rate limiting:**

```bash
# Automatic exponential backoff handles most rate limiting
# For persistent issues, try local subtitle files
episcan /path/to/videos --subtitles-dir /path/to/subtitles
```

## Dependencies

- `subliminal>=2.1.0` - Multi-provider subtitle downloads with dynamic discovery
- `pysubs2>=1.6.0` - Universal subtitle parsing
- `platformdirs>=3.0.0` - OS-appropriate cache directories
- `sentence-transformers` - Text similarity embeddings
- `openai-whisper` - Audio transcription
- `tmdbsimple` - TMDB API client
- `tvdb-v4-official` - TVDB API client
- `scipy` - Hungarian algorithm for optimal matching
- `torch` - GPU acceleration support

## License

MIT License - see LICENSE file for details.
