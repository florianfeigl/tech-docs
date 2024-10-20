+++
title = 'Automating Script Managment'
date = 2024-10-20T23:01:29+02:00
tags = ["bash", "software", "scripting", "automation"]
+++
### Automating Script Management for Local Use

Recently, I’ve begun automating a few small processes, such as cloud synchronization and backup creation. These tasks help me streamline my workflows and reduce the manual effort involved in keeping my system up to date. To make the automation scripts conveniently available locally, I’ve written a small Bash script that allows me to easily link any new scripts I create to a directory in my system's `$PATH`, making them accessible from anywhere in the terminal.

This script creates symbolic links to my scripts, so I can simply type the name of the script in the terminal to execute it, without having to navigate to its directory. Special thanks to ChatGPT for helping me with the creation of this script!

### The Script

Here’s a quick look at the script I’ve created:

```bash
#!/usr/bin/env bash

for file in "$HOME/src/Grive/Scripts"/*; do 
  if [ -f "$file" ]; then
    filename=$(basename "$file")
    link="/usr/local/bin/${filename%.*}"

    # Check if the symlink or the destination file already exists
    if [ -e "$link" ]; then
      echo "Symlink or file '$link' already exists. Skipping..."
    else
      sudo ln -s "$file" "$link"
      echo "Created symlink: $link -> $file"
    fi
  fi
done
```

### How It Works:
- **Iterate Over Files**: The script loops through all files in the `~/src/Grive/Scripts` directory.
- **Symlink Creation**: For each file, a symbolic link (symlink) is created in `/usr/local/bin/`, which is typically included in the system’s `$PATH`. This allows me to run the scripts from anywhere by just typing their names.
- **Name Handling**: The script strips the file extension using `${filename%.*}` to ensure the symlink has a clean, user-friendly name.
- **Check for Existing Files**: Before creating a symlink, the script checks if a file or symlink with the same name already exists in `/usr/local/bin`. If it does, the script skips the creation and moves on to the next file, avoiding overwrites.
- **Convenience**: I only need to run this script whenever I create a new automation script, and it automatically makes the new scripts globally accessible.

### How to Use It

If you want to do something similar, here’s a quick how-to:

1. **Create a Directory for Your Scripts**:
   - Store your automation scripts in a folder, e.g., `~/src/Grive/Scripts`.

2. **Write the Bash Script**:
   - Copy the above script into a new file, such as `link-scripts.sh`.

3. **Make the Script Executable**:
   - Run `chmod +x link-scripts.sh` to make the script executable.

4. **Run the Script**:
   - Simply execute `./link-scripts.sh` whenever you add a new script to your `Scripts` directory. The new script will then be accessible from anywhere in your terminal!

With this small automation, I’ve simplified the process of using my scripts. No more hunting for the right directory — I can just run the script name from the command line!
