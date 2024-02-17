# **Decluttarr**

## Overview
Decluttarr keeps the radarr & sonarr & lidarr queue free of stalled / redundant downloads.

Feature overview:
- Automatically delete downloads that are stuck downloading metadata (& trigger download from another source)
- Automatically delete failed downloads (& trigger download from another source)
- Automatically delete downloads belonging to Movies/TV shows/Music requests that have been deleted in the meantime ('Orphan downloads')
- Automatically delete stalled downloads, after they have been found to be stalled multiple times in a row
- Automatically delete downloads belonging to Movies/TV shows/Music requests that are unmonitored

You may run this locally by launching main.py, or by pulling the docker image.
You can find a sample docker-compose.yml in the docker folder.

## Dependencies
Use Sonarr v4 & Radarr v5 (currently 'nightly' tag instead of 'latest'), else certain features may not work correctly.
Use latest version of qBittorrent.

## Getting started
There's two ways to run this:
- As a docker container with docker-compose
- By cloning the repository and running the script manually

Both ways are explained below and there's an explanation for the different settings below that.

## Docker
1) Make a `docker-compose.yml` file
2) Use the following as a base for that and tweak the settings to your needs
```
version: "3.3"
services:
  decluttarr:
    image: ghcr.io/manimatter/decluttarr:latest
    container_name: decluttarr
    restart: always
    environment:
      - TZ=Europe/Zurich
      - PUID=1000
      - PGID=1000
      # General
      - LOG_LEVEL=INFO
      #- TEST_RUN=True 
      # Features 
      - REMOVE_TIMER=10
      - REMOVE_FAILED=True
      - REMOVE_STALLED=True
      - REMOVE_METADATA_MISSING=True     
      - REMOVE_ORPHANS=True
      - REMOVE_UNMONITORED=True
      - REMOVE_SLOW=True
      - MIN_DOWNLOAD_SPEED=100
      - PERMITTED_ATTEMPTS=3
      - NO_STALLED_REMOVAL_QBIT_TAG=Don't Kill
      # Radarr
      - RADARR_URL=http://radarr:7878
      - RADARR_KEY=$RADARR_API_KEY
      # Sonarr
      - SONARR_URL=http://sonarr:8989
      - SONARR_KEY=$SONARR_API_KEY
      # Lidarr
      - LIDARR_URL=http://lidarr:8686
      - LIDARR_KEY=$LIDARR_API_KEY
      # qBittorrent
      - QBITTORRENT_URL=http://qbittorrent:8080
      #- QBITTORRENT_USERNAME=Your name
      #- QBITTORRENT_PASSWORD=Your password
```
3) Run `docker-compose up -d` in the directory where the file is located to create the docker container

## Running manually
1) Clone the repository with `git clone https://github.com/Fxsch/decluttarr.git`
2) Rename the `config.conf-Example` inside the config folder to `config.conf`
3) Tweak `config.conf` to your needs
4) Install the libraries listed in the docker/requirements.txt (pip install -r requirements.txt)
5) Run the script with `python3 main.py`
Note: The `config.conf` is disregarded when running via docker-compose.yml

## Explanation of the settings
**LOG_LEVEL**
- Sets the level at which logging will take place
- `INFO` will only show changes applied to radarr/sonarr/lidarr
- `VERBOSE` will show when script runs (even if it results in no change)
- Type: String
- Permissible Values: CRITICAL, ERROR, WARNING, INFO, VERBOSE, DEBUG
- Is Mandatory: No (Defaults to INFO)

**TEST_RUN**
- Allows you to safely try out this tool. If active, downloads will not be removed.
- Type: Boolean
- Permissible Values: True, False
- Is Mandatory: No (Defaults to False)

---

### **Features settings**
- Steers which type of cleaning is applied to the downloads queue

**REMOVE_TIMER**
- Sets the frequency of how often the queue is checked for orphan and stalled downloads
- Type: Integer
- Unit: Minutes
- Is Mandatory: No (Defaults to 10)

**REMOVE_FAILED**
- Steers whether failed downloads with no connections are removed from the queue
- These downloads are not added to the blocklist
- Type: Boolean
- Permissible Values: True, False
- Is Mandatory: No (Defaults to False)

**REMOVE_STALLED**
- Steers whether stalled downloads with no connections are removed from the queue
- These downloads are added to the blocklist, so that they are not re-requested in the future
- A new download from another source is automatically added by radarr/sonarr/lidarr (if available)
- Type: Boolean
- Permissible Values: True, False
- Is Mandatory: No (Defaults to False)

**REMOVE_METADATA_MISSING**
- Steers whether downloads stuck obtaining metadata are removed from the queue
- These downloads are added to the blocklist, so that they are not re-requested
- A new download from another source is automatically added by radarr/sonarr/lidarr (if available)
- Type: Boolean
- Permissible Values: True, False
- Is Mandatory: No (Defaults to False)

**REMOVE_ORPHANS**
- Steers whether orphan downloads are removed from the queue
- Orphan downloads are those that do not belong to any requested media anymore (Since the media was removed from radarr/sonarr/lidarr after the download started)
- These downloads are not added to the blocklist
- Type: Boolean
- Permissible Values: True, False
- Is Mandatory: No (Defaults to False)

**REMOVE_UNMONITORED**
- Steers whether downloads belonging to unmonitored media are removed from the queue
- Note: Will only remove from queue if all TV shows depending on the same download are unmonitored
- These downloads are not added to the blocklist
- Note: Since sonarr does not support multi-season packs, if you download one you should protect it with `NO_STALLED_REMOVAL_QBIT_TAG` that is explained further down
- Type: Boolean
- Permissible Values: True, False
- Is Mandatory: No (Defaults to False)

**REMOVE_MISSING_FILES**
- Steers whether downloads that have the warning "Files Missing" are removed from the queue
- These downloads are not added to the blocklist
- Type: Boolean
- Permissible Values: True, False
- Is Mandatory: No (Defaults to False)

**REMOVE_SLOW**
- Steers whether slow downloads are removed from the queue
- Slow downloads are added to the blocklist, so that they are not re-requested in the future
- A new download from another source is automatically added by sonarr/radarr (if available)
- Type: Boolean
- Permissible Values: True, False
- Is Mandatory: No (Defaults to False)

**MIN_DOWNLOAD_SPEED**
- Sets the minimum download speed for active downloads
- If the increase in the downloaded file size of a download is less than this value between two consecutive checks, the download is considered slow and is removed if happening more ofthen than the permitted attempts.
- Type: Integer
- Unit: KBytes per second
- Is Mandatory: No (Defaults to 100, but is only enforced when "REMOVE_SLOW" is true)

**PERMITTED_ATTEMPTS**
- Defines how many times a download has to be caught as stalled, slow or stuck downloading metadata before it is removed
- Type: Integer
- Unit: Number of scans
- Is Mandatory: No (Defaults to 3)

**NO_STALLED_REMOVAL_QBIT_TAG**
- Downloads in qBittorrent tagged with this tag will not be killed even if they are stalled
- Also protects slow downloads or those stuck downloading meta data
- Tag is automatically created in qBittorrent (required qBittorrent is reachable on `QBITTORRENT_URL`)
- Also protects unmonitored downloads from being removed (relevant for multi-season packs)
- Type: String
- Is Mandatory: No (Defaults to `Don't Kill`)

---

### **Radarr section**
- Defines radarr instance on which download queue should be decluttered

**RADARR_URL**
- URL under which the instance can be reached
- If not defined, this instance will not be monitored

**RADARR_KEY**
- Your API key for radarr

---

### **Sonarr section**
- Defines sonarr instance on which download queue should be decluttered

**SONARR_URL**
- URL under which the instance can be reached
- If not defined, this instance will not be monitored

**SONARR_KEY**
- Your API key for sonarr

---

### **Lidarr section**
- Defines lidarr instance on which download queue should be decluttered

**LIDARR_URL**
- URL under which the instance can be reached
- If not defined, this instance will not be monitored

**LIDARR_KEY**
- Your API key for lidarr

---

### **qBittorrent section**
- Defines settings to connect with qBittorrent

**QBITTORRENT_URL**
- URL under which the instance can be reached
- If not defined, the NO_STALLED_REMOVAL_QBIT_TAG takes no effect

**QBITTORRENT_USERNAME**
- Username used to log in to qBittorrent
- Optional; not needed if authentication bypassing on qBittorrent is enabled (for instance for local connections)

**QBITTORRENT_PASSWORD**
- Password used to log in to qBittorrent
- Optional; not needed if authentication bypassing on qBittorrent is enabled (for instance for local connections)

## Dependencies
Use Sonarr v4 & Radarr v5 (currently 'nightly' tag instead of 'latest'), else certain features may not work correctly.
Use latest version of qBittorrent.

## Credits
- Script for detecting stalled downloads expanded on code by MattDGTL/sonarr-radarr-queue-cleaner
- Script to read out config expanded on code by syncarr/syncarr 
- SONARR/RADARR team & contributors for their great product, API documenation, and guidance in their Discord channel
- Particular thanks to them for adding an additional flag to their API that allowed this script detect downloads stuck finding metadata
- craggles17 for arm compatibility
- Fxsch for improved documentation / ReadMe

## Disclaimer
This script comes free of any warranty, and you are using it at your own risk.
