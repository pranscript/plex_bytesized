# Troubleshoot

1. How to handle gibberish filenames through Nzbget?
   1. You can use Videosort.py extension.
2. How to put Music, 1080p movies and 4K movies inside different folders?
   1. You can make categories for each thing inside settings->categories in Nzbget.
   2. Now, In Sonarr, Radarr or Lidarr, you can set the same category while setting up Nzbget
      1. Put Music in Lidarr, so Nzbget will download all requests from Lidarr to Music folder
      2. Put Movies in Radarr, so Nzbget will download all requests from Radarr to Movies folder
      3. Put Movies4K in another instance of Radarr, so Nzbget will download all requests from Radarr4K instance to Movies4K folder
      4. Put Series in Sonarr, so Nzbget will download all requests from Sonarr to Series folder.
3. How To import SRT files too while downloading?
   1. Radarr and Sonarr can automatically do this if you set the settings under media management.
4. How to manage huge library?
   1. Give tags while downloading anything. This helps to move a group of files together or if in future we need to change the quality profile or language.
   2. Move or delete files through Radarr or Sonarr only instead of manually deleting.
5. What RAM usage is normal?
   1. For my huge library, 300-400MB for Radarr and Sonarr is normal. 100-200 for Deluge and Jackett. 
   2. At the time of writing, Plex does have official memory leak, which creeps in as days passes by. So you might need to restart only plex every 4-5 days.
6.  