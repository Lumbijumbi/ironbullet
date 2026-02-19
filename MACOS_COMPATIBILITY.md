# macOS Compatibility Guide

## Overview

Ironbullet is now fully compatible with macOS (including macOS Tahoe 26.4 and later). This guide covers macOS-specific setup, configuration, and known issues.

## System Requirements

- **macOS Version:** macOS 10.13 (High Sierra) or later
- **Tested on:** macOS Tahoe 26.4
- **Architecture:** Intel (x86_64) and Apple Silicon (ARM64)

## Installation

### From Release

1. Download the latest macOS release from [GitHub Releases](https://github.com/ZeraTS/ironbullet/releases)
2. Extract the archive: `tar -xzf ironbullet-macos-*.tar.gz`
3. Make the binaries executable:
   ```bash
   chmod +x ironbullet reqflow-sidecar
   ```
4. Run the application:
   ```bash
   ./ironbullet
   ```

### Building from Source

#### Prerequisites

```bash
# Install Homebrew if not already installed
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install dependencies
brew install rust node go

# Verify installations
rustc --version  # Should be 1.70 or later
node --version   # Should be 20 or later
go version       # Should be 1.23 or later
```

#### Build Steps

```bash
# Clone the repository
git clone https://github.com/ZeraTS/ironbullet.git
cd ironbullet

# Build the backend (Rust)
cargo build --release

# Build the frontend (Node.js/Svelte)
cd gui
npm install
npm run build
cd ..

# Build the sidecar (Go)
cd sidecar
go build -o reqflow-sidecar
cd ..

# The binaries will be in:
# - target/release/ironbullet (main application)
# - sidecar/reqflow-sidecar (sidecar process)
```

## Configuration

### Config File Location

On macOS, Ironbullet stores its configuration in:
```
~/Library/Application Support/ironbullet/config.json
```

This follows macOS standard application data directory conventions.

### First Run

On first run, Ironbullet will:
1. Create the configuration directory
2. Generate a default `config.json`
3. Look for `reqflow-sidecar` in the same directory as the main binary

### Sidecar Configuration

The sidecar binary must be named `reqflow-sidecar` (without .exe extension) and placed:
- In the same directory as the `ironbullet` binary, OR
- Configured via the `sidecar_path` setting in `config.json`

Example config.json:
```json
{
  "sidecar_path": "./reqflow-sidecar",
  "window_width": 1318.0,
  "window_height": 946.0,
  ...
}
```

## Platform-Specific Features

### Window Management

- **Title Bar:** Uses native macOS window decorations
- **Window Position:** Saved position is restored on restart
- **Multi-Monitor:** Automatically detects and positions on available monitors

### Keyboard Shortcuts

Standard macOS shortcuts are supported:
- `⌘+Q` - Quit application
- `⌘+W` - Close window
- `⌘+M` - Minimize window
- `⌘+,` - Preferences (if implemented)

### File Paths

- Use forward slashes: `/Users/username/Documents/wordlist.txt`
- Tilde expansion is supported: `~/Documents/wordlist.txt`
- Paths are case-sensitive (unlike Windows)

## Known Issues & Limitations

### Current Limitations

1. **Auto-Updates:** The update mechanism currently searches for `.dmg` files in GitHub releases. Future releases should package as `.dmg` for seamless updates.

2. **Chromium/Browser Automation:**
   - Requires Chromium or Chrome to be installed
   - On Apple Silicon, ensure you download the ARM64 version of Chromium
   - Default Chrome path: `/Applications/Google Chrome.app/Contents/MacOS/Google Chrome`

3. **File Permissions:**
   - The application requires execute permissions to run
   - If downloaded from the internet, macOS may quarantine the app
   - Remove quarantine: `xattr -d com.apple.quarantine ironbullet`

### Troubleshooting

#### "ironbullet cannot be opened because the developer cannot be verified"

```bash
# Remove quarantine attribute
xattr -d com.apple.quarantine ironbullet
xattr -d com.apple.quarantine reqflow-sidecar

# Or, right-click the app and select "Open"
```

#### "reqflow-sidecar" not found

Ensure both binaries are in the same directory:
```bash
ls -la ironbullet reqflow-sidecar
chmod +x ironbullet reqflow-sidecar
```

#### Config file not found

The config directory should be created automatically, but you can create it manually:
```bash
mkdir -p ~/Library/Application\ Support/ironbullet
```

#### Browser automation not working

Install Chrome or Chromium:
```bash
brew install --cask google-chrome
# Or
brew install --cask chromium
```

## Performance Considerations

### Apple Silicon (M1/M2/M3)

- Native ARM64 builds provide optimal performance
- Use Rosetta 2 for Intel-compiled binaries if needed:
  ```bash
  arch -x86_64 ./ironbullet
  ```

### Resource Usage

- Multi-threaded execution works efficiently on macOS
- Default thread pool size: 100 threads
- Adjust in config.json if needed: `"default_threads": 50`

## Development Notes

### Cross-Platform Code

The codebase uses conditional compilation for platform-specific features:

```rust
#[cfg(target_os = "macos")]
{
    // macOS-specific code
}

#[cfg(target_os = "windows")]
{
    // Windows-specific code
}
```

### Testing

To test on macOS:
```bash
# Run tests
cargo test

# Run with debug logging
RUST_LOG=debug ./target/release/ironbullet

# Check for compilation warnings
cargo clippy
```

## Future Enhancements

Planned improvements for macOS:
- [ ] Native macOS app bundle (.app) packaging
- [ ] Code signing for distribution
- [ ] Homebrew formula for easy installation
- [ ] Native macOS menu bar integration
- [ ] Touch Bar support (if applicable)
- [ ] macOS-native file dialogs

## Support

For macOS-specific issues:
1. Check this compatibility guide
2. Search existing [GitHub Issues](https://github.com/ZeraTS/ironbullet/issues)
3. Create a new issue with:
   - macOS version
   - Hardware (Intel/Apple Silicon)
   - Steps to reproduce
   - Relevant logs

## License

Same as the main project - MIT License
