# RCurrent: Minimalist Right-Aligned Zsh Prompt with Battery & Time Awareness

A clean, extensible right-prompt (`RPROMPT`) system inspired by Agnoster's segment design. Features intelligent battery status visualization and date-aware time display – all in a single-file configuration.

![RCurrent Prompt Preview](https://via.placeholder.com/600x100/2d2d2d/ffffff?text=🔋85%25%E2%96%B2+%7C+%E2%9A%A1+14%3A30%3A45+%7C+%F0%9F%98%8A+100%25+%7C+%F0%9F%95%92+2024-06-15+09%3A00%3A01)

> **Note**: Actual appearance depends on your terminal's color scheme and [Nerd Fonts](https://www.nerdfonts.com/) support. Placeholder image shows conceptual layout.

## ✨ Key Features

- **Battery-aware visualization**:
  - `🔋`/`🪫` icons with color-coded levels (green → red)
  - State indicators: `▲` (charging), `▼` (discharging), `😊` (full), `⚠️` (unknown)
  - Automatic desktop detection (no output on non-battery devices)
- **Smart time display**:
  - Shows full timestamp (`2024-06-15 09:00:01`) only on date change
  - Compact time (`14:30:45`) for subsequent commands same day
  - Persistent state via `~/.zsh-last-cmd-date`
- **Segment-based architecture**:
  - `prompt_rsegment()` for building colored segments
  - `prompt_rend()` for clean segment termination
  - Easily extensible for custom right-prompt modules
- **RPROMPT-safe rendering**:
  - Proper `%F{}/%K{}` sequences for correct width calculation
  - No prompt corruption in tmux/screen/VS Code terminals
- **macOS-optimized**:
  - Uses native `pmset` for battery status (no external dependencies)
  - Works on Apple Silicon and Intel Macs

## ⚙️ Installation

### 1. Prerequisites
- **Zsh 5.0.3+** (macOS default is sufficient)
- **Nerd Fonts** (for powerline separators ``/``):
  ```bash
  # Recommended: Install Cascadia Code PL
  brew tap homebrew/cask-fonts
  brew install --cask font-cascadia-code-pl