# Episcan

Match TV episodes using audio transcription and episode subtitles (default) or descriptions.

## Features

- **Audio Transcription**: Uses OpenAI Whisper to transcribe video audio (loaded on-demand)
- **Time-Synchronized Comparison**: Compares matching time segments for fair subtitle-to-subtitle comparison
- **Smart Subtitle Track Selection**: Finds best matching video subtitle track using similarity analysis
- **Comprehensive Subtitle Caching**: OS-appropriate cache storage with automatic management
- **Multiple Subtitle Sources**:
  - Local subtitle files (`--subtitles-dir`)
  - Embedded video subtitles (`--try-subtitles`)
  - Subliminal library (default) - Dynamic provider discovery, no API key needed
- **Intelligent Transcription Defaults**:
  - Subtitle comparison: 3 minutes starting at 1 minute (skips intros)
  - Description comparison: Full episode transcription
- **Universal Subtitle Support**: Handles SRT, WebVTT, ASS/SSA, MicroDVD, MPL2, TMP, and JSON formats via pysubs2
- **Optimal Episode Matching**: Uses SBERT embeddings and Hungarian algorithm for unique assignments
- **GPU Acceleration**: CUDA support for both Whisper and sentence transformers
- **Memory Efficient**: Conditional model loading saves resources when not needed
- **Progress Tracking**: ETA calculations and detailed processing feedback
- **Multiple APIs**: TMDB (preferred) and TVDB support

## Quick Start

```bash
# Default: Use subliminal for subtitle downloads with caching, 3-minute excerpt comparison
uv run python main.py /path/to/videos

# Use local subtitle files
uv run python main.py /path/to/videos --subtitles-dir /path/to/subtitles

# Compare against episode descriptions instead
uv run python main.py /path/to/videos --use-descriptions

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
export TMDB_API_KEY="your_tmdb_key"      # Preferred
export TVDB_API_KEY="your_tvdb_key"      # Fallback
```

## How It Works

### Subtitle Comparison (Default)

1. **Check Cache**: Looks for previously downloaded subtitles in OS-appropriate cache directory
2. **Download Subtitles**: Uses subliminal with dynamic provider discovery for episode subtitles
3. **Cache Storage**: Saves downloaded subtitles with metadata for future use
4. **Time-Synchronized Extraction**: For partial transcription, extracts matching time segments from episode subtitles
5. **Smart Track Selection**: When using `--try-subtitles`, finds video subtitle track with best similarity to episode content
6. **Fair Comparison**: Compares equivalent content (3min transcript vs 3min subtitle segment)
7. **Optimal Assignment**: Uses Hungarian algorithm to ensure unique episode matches

### Description Comparison

1. **Full Transcription**: Transcribes entire episodes for comprehensive comparison
2. **Metadata Matching**: Compares transcripts against episode descriptions from TMDB/TVDB
3. **Similarity Scoring**: Uses cosine similarity with sentence transformers

## Subtitle Sources

**Priority Order:**

1. **Local Files** (`--subtitles-dir`) - Custom subtitle directory
2. **Subliminal** (default) - Dynamic provider discovery with caching:
   - Automatically detects all available providers
   - Intelligent fallback strategy (all → reliable → OpenSubtitles only)
   - OS-appropriate cache storage (macOS: `~/Library/Caches/episcan/`)
   - Cache hits eliminate re-downloads for repeated processing

## Why Subliminal?

- **Zero Configuration**: No API keys required
- **Dynamic Provider Discovery**: Automatically uses all available providers
- **High Success Rate**: Intelligent fallback strategy increases subtitle availability
- **Smart Caching**: Prevents re-downloads with persistent storage
- **Format Support**: Handles various subtitle formats automatically
- **Respectful**: Built-in rate limiting and provider rotation

## Command Line Options

```
positional arguments:
  video_dir             Directory containing video files (default: current directory)

options:
  --tvdb-api-key        TVDB API key (or use TVDB_API_KEY environment variable)
  --tmdb-api-key        TMDB API key (or use TMDB_API_KEY environment variable)
  --subtitles-dir       Directory containing subtitle files for episodes
  --use-descriptions    Use episode descriptions instead of subtitles (default: use subtitles)
  --force-tvdb          Force TVDB even if TMDB key available
  --whisper-model       Whisper model: tiny, base, small, medium, large (default: base)
  --sbert-model         Sentence transformer model (default: all-mpnet-base-v2)
  --max-duration        Transcription duration in seconds (default: 180 for subtitles, 0=full)
  --start-offset        Skip intro seconds (default: 60)
  --rename              File renaming: none, prompt, auto (default: none)
  --try-subtitles       Try embedded subtitles first, fallback to Whisper
  --clear-cache         Clear subtitle cache before processing
  --no-cache            Disable subtitle caching (always download fresh)
  --verbose             Detailed processing information
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
Loading sentence transformer model (sentence-transformers/all-mpnet-base-v2) on cuda...
Loading Whisper model on cuda...
Found 8 video files
Detected: Breaking Bad Season 1
Fetching episode subtitles using subliminal...
Found 8 episodes for Breaking Bad Season 1
Processing 8 video files...
  1/8: episode1.mkv ✓ (12.3s)
  2/8: episode2.mkv ✓ (11.8s)
  ...

Calculating optimal matches...

=== FINAL MATCHES ===
✓ episode1.mkv -> S01E01 - Pilot
  Similarity: 0.847
  Method: sbert_similarity

→ episode5.mkv -> S01E05 - Gray Matter
  Similarity: 0.723
  Method: sbert_similarity
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

**No subtitles found:**

```bash
# Install subliminal if missing
uv add subliminal

# Use description comparison as fallback
episcan /path/to/videos --use-descriptions
```

**Poor matching accuracy:**

```bash
# Try full episode transcription
episcan /path/to/videos --max-duration 0

# Use higher quality models
episcan /path/to/videos --whisper-model medium --sbert-model sentence-transformers/all-mpnet-base-v2
```

**API Rate Limiting:**

```bash
# Use longer delays (subliminal handles this automatically)
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
