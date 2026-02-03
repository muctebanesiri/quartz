---
title: Sent Presentation Tool with RTL and PDF Support
description: A fork of [fork](https://github.com/2qar/sent-pdf) of [suckless sent](https://tools.suckless.org/sent/) minimal presentation tool with full Right-to-Left (RTL) language support and PDF export capabilities.
keywords: presaentation, sent, suckless
draft: true
tags:
  - sent
  - presentation
date:
---
A fork of [fork](https://github.com/2qar/sent-pdf) of [suckless sent](https://tools.suckless.org/sent/) minimal presentation tool with full Right-to-Left (RTL) language support and PDF export capabilities.

![banner](https://github.com/user-attachments/assets/6ec07d61-8596-4ec4-9319-ac72b22ca37f)


## Features

- **Original sent functionality**: Lightweight, keyboard-driven presentations
- **RTL language support**: Full support for Persian, Arabic,Hebrew , and other RTL languages
- **PDF export**: Generate PDF versions of your presentations with a single keypress
- **Enhanced font support**: Pre-configured for optimal RTL text rendering with Vazirmatn and other fonts
- **FriBiDi integration**: Proper text shaping and bidirectional text rendering

## Installation

### Dependencies

```bash
# Ubuntu/Debian
sudo apt install libx11-dev libxft-dev libxinerama-dev libfribidi-dev libcairo2-dev libfreetype6-dev

# Fedora
sudo dnf install libX11-devel libXft-devel libXinerama-devel fribidi-devel cairo-devel freetype-devel

# Arch Linux
sudo pacman -S libx11 libxft libxinerama fribidi cairo freetype2
```

### Building from Source

```bash
git clone https://github.com/muctebanesiri/sent.git
cd sent
make
sudo make install
```

## Usage

### Basic Presentation

```bash
sent presentation.txt
```

### Fullscreen Mode

```bash
sent -f presentation.txt
```

### Keyboard Controls

- **Arrow keys**: Navigate between slides
- **Space/Enter**: Next slide
- **Backspace**: Previous slide
- **g**: Generate PDF version
- **r**: Reload presentation file
- **q**: Quit
- **F11**: Toggle fullscreen

### PDF Generation

Press `g` during your presentation to generate a PDF version. The PDF will be saved as `[filename].pdf` in the current directory.

## RTL Language Support

This fork includes full support for Right-to-Left languages:

- Proper text rendering for Arabic, Hebrew, Persian, etc.
- FriBiDi library integration for bidirectional text support
- Pre-configured font fallbacks with Vazirmatn as the primary Arabic font

### Customizing Fonts

Edit `config.h` to customize font preferences:

```c
static char *fontfallbacks[] = {
	"Vazirmatn:style=Regular",
	"Noto Sans Arabic:style=Regular",
	"Aref Ruqaa:style=Bold",
	"iransansx",
	"roboto",
	"ubuntu",
};
```

## Presentation Format

Sent uses a simple text format for presentations:
see example file for more

```
# Slide 1 Title
Slide content
with multiple lines

# Slide 2 Title
More content
- With bullet points
- And more elements
```

### Special Features

- Lines starting with `#` are treated as slide separators
- Lines starting with `@` are treated as embedded images
- Use `\\` to escape special characters

## Customization

### Modifying Key Bindings

Edit the `shortcuts` array in `config.h`:

```c
static Shortcut shortcuts[] = {
	{ XK_Escape,      quit,           {0} },
	{ XK_g,           pdf,            {0} },
	// Add your custom shortcuts here
};
```

### Changing Colors

Modify the `colors` array in `config.h`:

```c
static const char *colors[] = {
	"#FFFFFF", /* foreground color */
	"#000000", /* background color */
};
```

## Troubleshooting

### Common Issues

1. **Fonts not rendering correctly**: Ensure required fonts are installed on your system
2. **PDF generation fails**: Verify Cairo library is properly installed
3. **RTL text not displaying properly**: Check FriBiDi library installation

### Debug Mode

Compile with debug flags for troubleshooting:

```bash
make clean
make DEBUG=1
```

## License

This project maintains the original sent license. See LICENSE file for details.

## Contributing

Contributions are welcome! Please feel free to submit pull requests or open issues for bugs and feature requests.

## Acknowledgments

- Original sent developers for the excellent minimal presentation tool
- FriBiDi library for RTL text support
- Cairo library for PDF export capabilities
- Vazirmatn font developers for the beautiful Arabic typeface

---

For more information, visit the [original sent page](https://tools.suckless.org/sent/) or the [GitHub repository](https://github.com/muctebanesiri/sent).
