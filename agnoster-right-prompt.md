# Right-Aligned Zsh Prompt with Battery & Time Awareness (macOS only)

A clean, extensible right-prompt (`RPROMPT`)  [Oh My Zsh](https://github.com/ohmyzsh/ohmyzsh) customization inspired by [Agnoster](https://github.com/agnoster/agnoster-zsh-theme) segment design. Features intelligent battery status visualization and date-aware time display – all in a single-file configuration.

## Previews

![Fully charged](fully-charged.jpg)
![Discharging](discharging.jpg)
![Charging](charging.jpg)
![Date Change](date-change.jpg)

## ✨ Key Features

NOTE: In all likelihood, you will need to install a [Powerline-patched font](https://github.com/powerline/fonts) for this theme to render correctly.

To test if your terminal and font support it, check that all the necessary characters are supported by copying the following command to your terminal: `echo "\ue0b0 \ue0b2 \u00b1 \ue0a0 \u27a6 \u2718 \u26a1 \u2699"`. The result should look like this:

![powerline-fonts](powerline-fonts.jpg)


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
- **macOS-only**:
  - Uses native `pmset` for battery status (no external dependencies)
  - Works on Apple Silicon and Intel Macs
  - Linux users can use platform commands to detect battery status. 

## ⚙️ Installation

### 1. Prerequisites
- **Zsh 5.0.3+** (macOS default is sufficient)
- **Nerd Fonts** or any [Powerline-patched font](https://github.com/powerline/fonts) (for powerline separators ``/``):
  ```bash
  # Recommended: Install Cascadia Code PL
  brew tap homebrew/cask-fonts
  brew install --cask font-cascadia-code-pl
  ```

_In iTerm2: Profiles → Text → Font → Select "Cascadia Code PL"_

### 2. Integration

Add this entire block to your `~/.zshrc` after `source $ZSH/oh-my-zsh.sh` and before any `RPROMPT` assignments:

```zsh
# ===== RCurrent Prompt System =====
# Inspired by Agnoster's segment design | github.com/yourname/rcurrent-prompt

# Color definitions (256-color compatible)
RCURRENT_BG='NONE'
RSEGMENT_SEPERATOR="\ue0b2"  # Powerline right separator ()
TIME_FG=0       # Black text
TIME_BG=15      # White background
POWER_BG=244    # Gray background
POWER_FG_GOOD=82      # Bright green (≥80%)
POWER_FG_FINE=green   # Green (60-80%)
POWER_FG_WARN=yellow  # Yellow (40-60%)
POWER_FG_CAUTION=208  # Orange (20-40%)
POWER_FG_BAD=196      # Red (<20%)
POWER_FG_UNKNOWN=244  # Gray (unknown state)

# Build right-aligned segment with background transition
prompt_rsegment() {
  local bg fg
  [[ -n $1 ]] && bg="%K{$1}" || bg="%k"
  [[ -n $2 ]] && fg="%F{$2}" || fg="%f"
  if [[ $RCURRENT_BG != 'NONE' && $1 != $RCURRENT_BG ]]; then
    echo -n " %F{$1}$RSEGMENT_SEPERATOR${bg}${fg} "
  else
    echo -n "%F{$1}$RSEGMENT_SEPERATOR${bg}${fg} "
  fi
  RCURRENT_BG=$1
  [[ -n $3 ]] && echo -n $3
}

# Terminate segment chain with proper color reset
prompt_rend() {
  [[ -n $RCURRENT_BG ]] && echo -n "%{%k%F{$RCURRENT_BG}%}" || echo -n "%{%k%}"
  echo -n "%{%f%}"
  RCURRENT_BG=''
}

# Time segment: Full timestamp on date change, compact time otherwise
prompt_time() {
  local state_file="${HOME}/.zsh-last-cmd-date"
  local cur_date=$(date +%Y-%m-%d)
  local cur_time_long=$(date '+%Y-%m-%d %H:%M:%S')
  local cur_time_short=$(date '+%H:%M:%S')
  local last_date="" && [[ -f "$state_file" ]] && last_date=$(<"$state_file")
  
  local display_time="$cur_time_short"
  [[ -z "$last_date" || "$cur_date" != "$last_date" ]] && display_time="$cur_time_long"
  
  # Atomic state update
  local tmp="${state_file}.tmp.$$"
  echo -n "$cur_date" >| "$tmp" 2>/dev/null && mv -f "$tmp" "$state_file" 2>/dev/null
  
  prompt_rsegment "$TIME_BG" "$TIME_FG" "⏲ ${display_time}"
}

# Battery segment: macOS-only (uses pmset)
prompt_power() {
  local batt_info && batt_info=$(pmset -g batt 2>/dev/null) || return 0
  [[ "$batt_info" == *"No battery"* || -z "$batt_info" ]] && return 0

  # Parse level and status
  local level="??"; [[ "$batt_info" =~ ([0-9]+)% ]] && level="${match[1]}"
  local pstatus="unknown"
  [[ "$batt_info" == *"charged"* ]] && pstatus="charged"
  [[ "$batt_info" == *"charging"* || "$batt_info" == *"finishing"* ]] && pstatus="charging"
  [[ "$batt_info" == *"discharging"* ]] && pstatus="discharging"

  # Icon selection (≥20% = 🔋, else 🪫)
  local icon="🪫"; [[ "$level" != "??" && $level -gt 20 ]] && icon="🔋"

  # State indicator and color
  local triangle="" status_color="$POWER_FG_UNKNOWN"
  case "$pstatus" in
    charged)   triangle="😊"; status_color="$POWER_FG_GOOD" ;;
    charging)  triangle="▲"; status_color="$POWER_FG_GOOD" ;;
    discharging) triangle="▼"; status_color="$POWER_FG_BAD" ;;
    *)         triangle="⚠️"; status_color="$POWER_FG_UNKNOWN" ;;
  esac

  # Level-based color
  local level_color="$POWER_FG_UNKNOWN"
  [[ "$level" != "??" ]] && {
    (( level >= 80 )) && level_color="$POWER_FG_GOOD" || 
    (( level >= 60 )) && level_color="$POWER_FG_FINE" || 
    (( level >= 40 )) && level_color="$POWER_FG_WARN" || 
    (( level >= 20 )) && level_color="$POWER_FG_CAUTION" || 
    level_color="$POWER_FG_BAD"
  }

  prompt_rsegment "$POWER_BG" "$level_color" "${icon}${level}%%%F{$status_color}${triangle}"
}

# Assemble right prompt segments
right_prompt() {
  prompt_time
  prompt_power
  prompt_rend
}

# Activate RPROMPT (must be LAST prompt-related setting in .zshrc)
RPROMPT='%{%f%b%k%}$(right_prompt)'
# ===== End RCurrent =====
```

### 3. Apply Changes

```bash
source ~/.zshrc
```