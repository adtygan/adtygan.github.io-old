+++
title = 'Never Miss a New File with fswatch'
date = '2024-03-28'
draft = false
+++

> 🔥 This is my first blog post • **LETS GO!**

## Context 
As someone with a new found interest in automating things, I have been working on building a local image search engine called **WarpSearch**. It will allow me to query locally stored images using text and this is amazing because I can dump all my images in one folder and be very hopeful that I will be able to find it later. But folders don't remain the same. Images get added or deleted and I need to keep track of these changes.

## Enter `fswatch`
This is where [`fswatch`](https://github.com/emcrisostomo/fswatch) comes in clutch. It's a tool that allows monitoring a folder for changes. And in this post, I want to cover a general recipe that allows for detecting file additions and deletions in a folder. 

Let's call the script `file-monitor.sh`. It monitors the folder path `~/Vault` and tracks files with extensions `.jpg`, `.jpeg` and `.png`. Below is the code:

```bash
FOLDER=~/Vault

# Define a function to get state of directory
get_state() {
  find $FOLDER -type f \( -iname "*.jpg" -o -iname "*.jpeg" -o -iname "*.png" \)
}

# Initialize previous_state with state of directory
previous_state=$(get_state)

# Monitor the ~/Vault directory for changes
fswatch -0 -o $FOLDER | while read -d "" event; do
  # For each event, list current state of directory 
  current_state=$(get_state)
  # Diff current state with previous state
  diff <(echo "$previous_state") <(echo "$current_state") | while read line; do
    if [[ $line == ">"* ]]; then
      echo "Added: ${line:2}"
    elif [[ $line == "<"* && ! -z "${line:2}" ]]; then
      echo "Deleted: ${line:2}"
    fi
  done
  previous_state=$current_state
done
```

At a high level, the code works by comparing the current state of the folder with its previous state. This allows it to detect file additions and deletions. After starting the script with `bash file-monitor.sh`, I added 2 images and renamed 1 exising image. Here is the output it produces:

```shell
Added: /Users/aditya/Vault/Orange Boat.jpg
Added: /Users/aditya/Vault/Futuristic Red.jpg
Deleted: /Users/aditya/Vault/Orange Boat.jpg
Added: /Users/aditya/Vault/Orange Boat Illustration.jpg
```

As you can see, the script handles renaming as a combination of deletion and addition.

