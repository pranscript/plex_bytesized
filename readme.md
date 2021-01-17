# Plex Server Setup 

Though many of you must have seen tremendously useful wikis and tutorials on how to set up, this is also one of those. But it is mainly focused on hosting on Bytesized-Hosting provider. I will also be linking most of the stuff to other tutorials from where I got the information. 

What is the end result?
  1. Rclone mount
  2. Plex server setup
  3. Radarr for Movies
  4. Sonarr for T.V Shows
  5. Nzbget for NZB handling 
  6. Deluge (Torrent)
  7. Jackett (Indexer)
  8. Ombi (User Requests)
  9. Bazarr (Subtitles)
  10. Filebot (Optional)

What it doesn't include and why?
  1. Couchpotato (since Radarr is much better)
  2. Sickbeard / Sickrage (since Sonarr is more stable)
  3. Olaris Rename (works great, but Filebot is supreme)
  4. Cardigann (Jackett works better)

# Pre-requisites
  - Bytesized account
  - SSH (Putty for Windows)
  - Filezilla (FTP application for easy handling of files)
  - Your own **Client I.D and Secret** for Rclone - You can see [this tutorial](https://quickbox.io/knowledge-base/creating-a-google-api-client-id-client-secret-code/).

# 1. Rclone

	1) Run "rclone config"
	2) press n
	3) Give name
	4) Choose 13 for Google Drive
	5) Put Client I.d
	6) Put Client Secret
	7) Put 1 (scope full access)
	8) Root folder default
	9) Service account default
	10) Choose 'N' for Advanced config
	11) Choose 'N' for Auto config (since we are in a headless machine) (a link will show, copy with highligting it and open in browser, login, click advanced, go to rclone, allow)
	12) Y (If it is a google team/shared drive)
	13) Write your team drive number if you selected "Y" above
	13) Y (after checking all is fine)

# 2. Mounting Drive and Mergerfs

We will follow this wonderful tutorial - [Bytesized Tutorial](https://bytesized-hosting.com/pages/setting-up-rclone-mergerfs-and-crontab-for-automated-cloud-storage). I did some few minor changes to Rclone parameters which I feel worked much better. So, let us continue. Rclone is used to mount the google drive and mergerfs fuses this drive to make flow of data from server to drive seamless. 
We will create the following folder through SSH on the server:

```sh
mkdir ~/mnt
mkdir ~/mnt/gdrive
mkdir ~/mnt/media_merge
mkdir ~/media_tmp
mkdir ~/scripts
mkdir ~/.config/mergerfs
```

You can follow the link mentioned above and create a startup and shutdown script if you want to use gcrypt (encrypt drive); otherwise, you can continue from here. I personally do not encrypt. Both the scripts are created so that while restarting the appbox (server), Rclone mount and mergerfs automatically starts on its own.

So, open a file using nano to create a startup script.

```sh
nano ~/.startup/gdrive
```

Paste the following code there (Copied from the tutorial link mentioned above)

- I have added a few additional parameters to Rclone, which works well for plex.
- Remember to replace **mountName** with your mount name, in Rclone execution code down below. Remove < > too.
- Remember to change **USER_ID, GROUP_ID, USER_AGENT** in the code below. First, two can be seen by typing **```id```** in terminal and copy the numeric numbers. (you can create another SSH session to find this if you are inside nano). USER_AGENT can be any random string. So let's make the startup script.

```sh
#!/bin/bash

USER_ID=XXXXX
GROUP_ID=XXXXX
USER_AGENT=XXXXXXXXXXXXXXXXX

export TMPDIR=$HOME/tmp
PID_FILE=$HOME/.config/rclone/rclone.pid
if [ -e $PID_FILE ]; then
    PID=`cat $PID_FILE`
    if ! kill -0 $PID > /dev/null 2>&1; then
        echo "Removing stale $PID_FILE"
        rm $PID_FILE
    fi
fi

/sbin/start-stop-daemon -S --pidfile $PID_FILE --make-pidfile -u $USER -d $HOME -b -a /usr/local/bin/rclone -- mount <mountName>: ~/mnt/gdrive --allow-other --user-agent="$USER_AGENT" --timeout 1h --dir-cache-time 1000h --cache-info-age 1500h  --poll-interval 15s --vfs-read-chunk-size 32M --buffer-size 32M --vfs-read-ahead 256M --vfs-cache-mode full --vfs-cache-max-size 200G --vfs-cache-max-age 336h --uid $USER_ID --gid $GROUP_ID

PID_FILE=$HOME/.config/mergerfs/mergerfs.pid
if [ -e $PID_FILE ]; then
    PID=`cat $PID_FILE`
    if ! kill -0 $PID > /dev/null 2>&1; then
        echo "Removing stale $PID_FILE"
        rm $PID_FILE
    fi
fi

/sbin/start-stop-daemon -S --pidfile $PID_FILE --make-pidfile -u $USER -d $HOME -b -a /usr/bin/mergerfs -- -f -o defaults,sync_read,auto_cache,use_ino,allow_other,func.getattr=newest,category.action=all,category.create=ff $HOME/media_tmp:$HOME/mnt/gdrive $HOME/mnt/media_merge
```

Save it.

Now create a shutdown script using similar process.

```sh
nano ~/.shutdown/gdrive
```
Paste the below code (Copied from the tutorial mentioned above)
```sh
#!/bin/bash

PID_FILE=$HOME/.config/rclone/rclone.pid
/sbin/start-stop-daemon --pidfile $PID_FILE -u $USER -d $HOME -K -a /usr/local/bin/rclone

if [ -e $PID_FILE ]; then
    PID=`cat $PID_FILE`
    if ! kill -0 $PID > /dev/null 2>&1; then
        echo "Removing stale $PID_FILE"
        rm $PID_FILE
    fi
fi

PID_FILE=$HOME/.config/mergerfs/mergerfs.pid
/sbin/start-stop-daemon --pidfile $PID_FILE -u $USER -d $HOME -K -a /usr/bin/mergerfs

if [ -e $PID_FILE ]; then
    PID=`cat $PID_FILE`
    if ! kill -0 $PID > /dev/null 2>&1; then
        echo "Removing stale $PID_FILE"
        rm $PID_FILE
    fi
fi
```
Now to make both the files executable, run the following command
```sh
chmod +x ~/.startup/gdrive
chmod +x ~/.shutdown/gdrive
```

Till now, we have made the script that will mount and unmount the drive while restarting the server.

Now create a script that will upload from Bytesized local drive to google drive.

As you can see, we have created a folder "media_tmp", which will store the downloaded files locally on Bytesized. We will move these files from this folder to our google drive and in the process, will delete the local.

```sh
#create a file inside scripts folder
nano ~/scripts/uploadmedia
# paste the below line there
screen -dmS uploadmedia /usr/local/bin/rclone move ~/media_tmp <mountName>: --delete-empty-src-dirs -P --stats 20s --log-file=rcloneMoviesLog.txt
# replace mountName that you put in rclone config and do not include < >
# Press Ctrl + X
# Press Y for yes.
# Press Enter to save and confirm file name.
chmod +x ~/scripts/uploadmedia  # to make the script executable.
```
We created the script to upload files. Now, we will create a **cron job **that will upload automatically, the content from local to our drive according to the script that we just made,at a specific time:

```sh
# open crontab
crontab -e
# Put below line in the file
0 5 * * * ~/scripts/uploadmedia
# Save it by pressing Ctrl + X,  Y and the Enter
# This will upload everyday at 5 a.m server time.
```

You can visit [crontab.guru](https://crontab.guru/) to understand the convention of setting up the time.

At the end, **restart** the appbox (server) and wait few minutes for it to mount (generally huge collection takes time to mount. Small drive will mount instantly).

To cross check if both rclone and mergerfs is working, you can check from the memory usage dashboard on Bytesized websute or you can run **```ps -ef```** in the terminal to see both are running. 

![GitHub Logo](./images/mergerfs.jpg)

# Utilities




| Order | Utility | Guide                                                        | Support                                                      |
| ----- | ------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 3.    | Nzbget  | [Nzbget](https://github.com/pranscript/plex_bytesized/tree/master/nzbget) | [Github](https://github.com/nzbget/nzbget) [Forum](https://forum.nzbget.net/) |
| 4.    | Deluge  | [Deluge](https://github.com/pranscript/plex_bytesized/tree/master/deluge) | [Github](https://github.com/deluge-torrent/deluge) [Forum](https://forum.deluge-torrent.org/) |
| 5.    | Jackett | [Jackett](https://github.com/pranscript/plex_bytesized/tree/master/jackett) | [Github](https://github.com/Jackett/Jackett) [Reddit](https://www.reddit.com/r/Jackett/) |
| 6.    | Radarr  | [Radarr](https://github.com/pranscript/plex_bytesized/tree/master/radarr) | [Discord](https://discord.gg/u3x3Kp8) [Github](https://github.com/Radarr/Radarr) [Reddit](https://www.reddit.com/r/radarr) |
| 7.    | Sonarr  | [Sonarr](https://github.com/pranscript/plex_bytesized/tree/master/sonarr) | [Discord](https://discord.gg/M6BvZn5) [Github](https://github.com/Sonarr/Sonarr) [Forum](https://forums.sonarr.tv/) [Reddit](https://www.reddit.com/r/sonarr) |
| 8.    | Plex    | [Plex](https://github.com/pranscript/plex_bytesized/tree/master/plex) | [Forum](https://forums.plex.tv/)                             |
| 9.    | Bazarr  | [Bazarr](https://github.com/pranscript/plex_bytesized/tree/master/bazarr) | [Discord](https://discord.com/invite/MH2e2eb) [Github](https://github.com/morpheus65535/bazarr) [Reddit](https://www.reddit.com/r/bazarr/) |
| 10.   | Ombi    | [Ombi](https://github.com/pranscript/plex_bytesized/tree/master/ombi) | [Discord](https://discord.gg/Sa7wNWb) [Github](https://github.com/tidusjar/Ombi) |
| 11.   | Filebot | [Filebot](https://github.com/pranscript/plex_bytesized/tree/master/filebot) | [Forum](https://www.filebot.net/forums/)                     |

# Extra


| Index | Topic                                                        |
| ----- | ------------------------------------------------------------ |
| 1.    | [Radarr 4K Instance](https://github.com/pranscript/plex_bytesized/tree/master/Extras/radarr4K.md) |
| 2.    | [SubZero bundle for Subtitles (Bazarr Alternative)](https://github.com/pranscript/plex_bytesized/tree/master/Extras/subzero.md) |
| 3.    | [OMBI Alternative - Requestrr for discord](https://github.com/darkalfx/requestrr) |
| 4.    | [Gclone, to copy between drives using Service accounts](https://gist.github.com/pranscript/731831c8f48a0ee8fdbd2acc646fb7e2) |



# Troubleshoot

[Common problems and fixes](https://github.com/pranscript/plex_bytesized/tree/master/troubleshoot)



<a href="https://www.buymeacoffee.com/pranscript" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/default-orange.png" alt="Buy Me A Coffee" height="41" width="174"></a>

