---
title: How to Run Custom Scripts in Nautilus
description: Learn how to add custom right-click actions to Nautilus (GNOME Files) in Linux. This step-by-step tutorial shows you how to create, install, and use automation scripts to batch convert images, rename files, and boost productivity.
keywords: nautilus scripts, linux file manager, custom scripts, right-click automation, gnome files, bash scripting, linux productivity, nautilus tutorial, file manager automation, image conversion avif
draft: false
tags:
  - Linux
  - Nautilus
  - Automation
  - Bash-Scripting
  - Productivity
  - Tutorial
  - GNOME
  - File-Manager
  - arch
date: 2026-02-03
---
Have you ever wished you could right-click on files in your Linux file manager and run custom scripts with a single click? What if you could convert images to modern formats, extract archives in special ways, or batch-rename files without ever opening a terminal? With Nautilus scripts, you can transform your standard file manager into a powerhouse of productivity.

In this guide, I'll explain how to create, install, and use custom scripts in Nautilus (also known as GNOME Files), turning repetitive tasks into one-click operations.

## What Are Nautilus Scripts?

Nautilus scripts are small executable programs that appear in your right-click context menu when you select files or folders. They're essentially shortcuts to automate tasks you frequently perform in your file manager. Unlike full applications, they're lightweight, easy to create, and integrate seamlessly into your workflow.

Think of them as custom tools you add to your file manager's toolbox. Once set up, they'll save you countless clicks and commands.

## Setting Up Your Scripts Directory

First, you need to create the directory where Nautilus looks for scripts. This is a hidden folder in your home directory.

Open your terminal and run:
```bash
mkdir -p ~/.local/share/nautilus/scripts/
```

The `-p` flag ensures all parent directories are created if they don't exist. This location works for most modern GNOME-based distributions.

## Creating Your First Script

Let's create a simple script to understand the basics. We'll make one that shows a notification with the paths of selected files.

1. **Create the script file**:
```bash
nano ~/.local/share/nautilus/scripts/show-file-paths
```

2. **Add this content**:
```bash
#!/bin/bash
# Simple script to show selected file paths

# Loop through each selected file
echo "$NAUTILUS_SCRIPT_SELECTED_FILE_PATHS" | while read file
do
    if [ -n "$file" ]; then
        # Show a desktop notification
        notify-send "Selected File" "$file"
    fi
done

# Also show a summary dialog
zenity --info --text="You selected $(echo "$NAUTILUS_SCRIPT_SELECTED_FILE_PATHS" | wc -l) file(s)"
```

3. **Make it executable**:
```bash
chmod +x ~/.local/share/nautilus/scripts/show-file-paths
```

**Understanding the script elements**:
- `#!/bin/bash` tells the system this is a Bash script
- `$NAUTILUS_SCRIPT_SELECTED_FILE_PATHS` is a special variable containing the paths of files you right-clicked
- `notify-send` creates a desktop notification
- `zenity` creates simple graphical dialogs
- `chmod +x` makes the file executable (required for all Nautilus scripts)

## Installing and Running Scripts

Once you've created a script and made it executable, you might need to restart Nautilus to see it in the menu:

```bash
nautilus -q
```

Then, simply:
1. Select one or more files in Nautilus
2. Right-click to open the context menu
3. Navigate to the "Scripts" submenu
4. Click your script name to run it

## Practical Example: My Image Converter Script

Let's create a truly useful script based on our previous discussion about image formats. This script will convert images to AVIF format with quality options.

Create a new file `convert-to-avif` in your scripts directory:

```bash
#!/bin/bash
# Convert selected images to AVIF format with quality options

# Get quality setting from user (AVIF scale: 30-80 is practical)
QUALITY=$(zenity --scale \
    --title="AVIF Conversion Quality" \
    --text="Select quality (30-80 recommended):\n30-40: High compression\n50-60: Balanced\n70-80: High quality" \
    --min-value=1 --max-value=100 --value=60)

if [ $? -ne 0 ]; then
    # User clicked Cancel
    exit 0
fi

# Counter for results
converted=0
failed=0

# Process each selected file
echo "$NAUTILUS_SCRIPT_SELECTED_FILE_PATHS" | while read file
do
    if [ -n "$file" ]; then
        # Check if it's an image file
        if file -b --mime-type "$file" | grep -q "^image/"; then
            
            # Create output filename
            base="${file%.*}"
            output="${base}.avif"
            
            # Check if output already exists
            if [ -f "$output" ]; then
                zenity --question \
                    --text="File $output already exists. Overwrite?" \
                    --title="File Exists"
                if [ $? -ne 0 ]; then
                    continue
                fi
            fi
            
            # Convert the image
            if convert "$file" -quality $QUALITY "$output" 2>/dev/null; then
                converted=$((converted + 1))
            else
                failed=$((failed + 1))
                echo "$file" >> /tmp/failed_avif_conversions.txt
            fi
        fi
    fi
done

# Show results
zenity --info \
    --title="Conversion Complete" \
    --text="Converted $converted image(s) to AVIF format.\nFailed: $failed"

if [ $failed -gt 0 ]; then
    zenity --text-info \
        --title="Failed Conversions" \
        --filename=/tmp/failed_avif_conversions.txt \
        --width=600 --height=400
    rm /tmp/failed_avif_conversions.txt
fi
```

Make it executable:
```bash
chmod +x ~/.local/share/nautilus/scripts/convert-to-avif
```

## Understanding Nautilus Environment Variables

When Nautilus runs your script, it provides useful information through environment variables:

| Variable | Description | Example |
|----------|-------------|---------|
| `NAUTILUS_SCRIPT_SELECTED_FILE_PATHS` | Newline-separated list of full paths to selected files | `/home/user/image.jpg`<br>`/home/user/document.pdf` |
| `NAUTILUS_SCRIPT_SELECTED_URIS` | URI-encoded versions of selected files (for remote locations) | `file:///home/user/image.jpg` |
| `NAUTILUS_SCRIPT_CURRENT_URI` | URI of current directory | `file:///home/user/Documents` |

## Pro Tips for Advanced Scripting

### 1. **Organize with Submenus**
Create subdirectories in your scripts folder to organize related scripts:
```bash
mkdir ~/.local/share/nautilus/scripts/Image\ Tools/
mkdir ~/.local/share/nautilus/scripts/Archive\ Tools/
```
Scripts in these folders will appear as submenus.

### 2. **Add Keyboard Shortcuts**
Edit the configuration file to add keyboard shortcuts:
```bash
nano ~/.config/nautilus/scripts-accels
```
Add lines like:
```
F2 convert-to-avif
<Control><Shift>R rename-files
```

### 3. **Create Multi-Language Scripts**
Nautilus scripts aren't limited to Bash! You can use Python, Perl, or any other language.

**Python example** (`resize-images.py`):
```python
#!/usr/bin/env python3
import os
from PIL import Image
import gi
gi.require_version('Gtk', '3.0')
from gi.repository import Gtk

# Get selected files from environment variable
files = os.environ.get('NAUTILUS_SCRIPT_SELECTED_FILE_PATHS', '').splitlines()

for file in files:
    if file.lower().endswith(('.png', '.jpg', '.jpeg')):
        img = Image.open(file)
        img.thumbnail((800, 800))
        base, ext = os.path.splitext(file)
        img.save(f"{base}_resized{ext}")
        
Gtk.MessageDialog(None, 0, Gtk.MessageType.INFO, 
                  Gtk.ButtonsType.OK, "Resized images").run()
```

### 4. **Handle Special Characters in Filenames**
Some files have spaces or special characters. Use proper quoting in your scripts:
```bash
#!/bin/bash
# Correct way to handle any filename
while IFS= read -r file
do
    echo "Processing: '$file'"
    # Always quote variables with filenames
    ls -la "$file"
done <<< "$NAUTILUS_SCRIPT_SELECTED_FILE_PATHS"
```

## Troubleshooting Common Issues

1. **Scripts don't appear in the menu**:
   - Ensure the script is executable (`chmod +x`)
   - Check it's in the correct directory
   - Restart Nautilus with `nautilus -q`

2. **Script runs but doesn't work properly**:
   - Check for errors: run the script from terminal with sample file paths
   - Ensure all required packages are installed
   - Add error logging to your script

3. **Dialogs don't appear**:
   - Some scripts need a display; they won't work over SSH without X11 forwarding
   - Use `zenity` or `notify-send` for GUI feedback


## The Complete Workflow In a Nutshell

To visualize the entire process from creation to execution:

1. **Create** your script in the `~/.local/share/nautilus/scripts/` directory
2. **Make it executable** with `chmod +x`
3. **Optionally organize** into subdirectories for better menu structure
4. **Restart Nautilus** if scripts don't appear immediately
5. **Use** by right-clicking files and selecting from the Scripts menu
6. **Debug** by running from terminal if needed