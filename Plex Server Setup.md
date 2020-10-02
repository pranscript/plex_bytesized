# (In Progress)
# Plex Server Setup 

Though many of you had saw tremendously good wiki and tutorials on how to setup, this is obviously one of those too. But it is mainly focused on hosting on Bytesied-Hosting provider. Also, I will be linking most of the stuffs to other tutorials from where I got the information. 

What is the end result?
  1. Rclone mount
  2. Plex server setup
  3. Radarr for Movies
  4. Sonarr for T.V Shows
  5. Nzbget and Sabnzbd for NZB handling 
  6. Deluge (Torrent)
  7. Jackett (Indexer)
  8. Ombi (User Requests)
  9. Bazarr (Subtitles)
  10. Filebot (Optional)

What it doesn't include and why?
  1. Couchpotato (since Radarr is much better)
  2. Sickbeard / Sickrage (since Sonarr is more stable)
  3. Olaris Rename (works great but Filebot is supreme)
  4. Cardigann (Jakett works better)

# Pre-requisites
  - Bytesized account
  - SSH (Putty for Windows)
  - Filezilla (FTP application for easy handling of files)

# Rclone

	1) Run "rclone config"
	2) press n
	3) Give name
	4) Choose 13 for Google Drive
	5) Put Clinet Id
	6) Put Client Secret
	7) Put 1 (scope full access)
	8) Root folder default
	9) Service account default
	10) Choose 'N' for Advanced config
	11) Choose 'N' for Auto config (since we are in a headless machine) (a link will show, copy with highligting it and open in browser, login, click advanced, go to rclone, allow)
	12) Y (If it is a google team/shared drive)
	13) Write your team drive number if you selected "Y" above
	13) Y (after checking all is fine)

# Mounting Drive using Rclone

We will follow [Bytesized Tutorial](https://bytesized-hosting.com/pages/setting-up-rclone-mergerfs-and-crontab-for-automated-cloud-storage)
We will create the following folder through Putty on server:
```sh
mkdir ~/mnt
mkdir ~/mnt/gdrive
mkdir ~/mnt/media_merge
mkdir ~/media_tmp
mkdir ~/scripts
mkdir ~/.config/mergerfs
```
Now create a script that will upload from local to google drive:
```sh
nano ~/scripts/uploadmedia
screen -dmS uploadmedia /usr/local/bin/rclone move ~/media_tmp <insertName>: --delete-empty-src-dirs -v --stats 5s   // name used in rclone config and remove < >
Press Ctrl + X
Press Y for yes.
Press Enter to confirm file name.
chmod +x ~/scripts/uploadmedia  //to make the script executable.
```
Now a crontab that will upload automatically at specific time:
```sh
crontab -e
0 5 * * * ~/scripts/uploadmedia // put this in the file
Press Ctrl + X,  Y and the Enter
```
Now follow the link and create a startup script:
```sh
Change the id, group id and user agent
--dir-cache-time 72h  --poll-interval 15s --vfs-read-chunk-size 16M // add these parameters to rclone too
chmod +x ~/.startup/gdrive
```

# Nzbget

We will use this to download movies.
