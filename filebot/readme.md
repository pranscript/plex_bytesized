# Filebot

### To rename a movie ,  the following command might be useful.

```sh
filebot -script fn:amc -rename "/home/hd*/username/media/Movies/" --output "/home/hd*/username/renamedMovies/" -r --action move --db TheMovieDB -non-strict --def excludeList=amc.txt "movieFormat={n.replaceAll(/[:|]/, ' - ')} ({y}) /{n.replaceAll(/[:|]/, ' - ')} ({y}) - {vf}.{vc.replace('x', 'h')}.{ac}.{af}{subt}"  --def clean=y --def subtitles=en --def "ut_label=Movies" --conflict auto > filebotMovie.txt
```

- ```fn:amc``` - The amc script is used to perform the operation
- ```-r``` - recursively go through the folders
- ```action move``` - After rename, move the file from source to destination.
- ```--db TheMovieDB``` - The movie database will be used to rename the files.
- ```--def excludeList=amc.txt``` -  After rename, this file will be populated so that next time, it will not rename anything present in this file.
- ```movieFormat={n.replaceAll(/[:|]/, ' - ')} ({y}) /{n.replaceAll(/[:|]/, ' - ')} ({y}) - {vf}.{vc.replace('x', 'h')}.{ac}.{af}{subt}``` - Example
  - Folder name -> The Movie Name (2020) 
  - Filename -> The Movie Name (2020) - 1080p.H264.AC3.6ch
  - Subtitle -> The Movie Name (2020) - 1080p.H264.AC3.6ch.en
- ```--def clean=y``` - This will delete all the empty folders after moving the renamed files.
- ```--def subtitles=en``` - This will download the subtitles according to the language set from opensubtitles if configured. 
- ```--conflict auto```  -  This will automatically resolve any conflicting cases related to file already present or movie names.
- ``` filebotMovie.txt```  - The output will be overwritten to this text file.



### To rename a series,  the following command might be useful.

```sh
filebot -script fn:amc -rename "/home/hd*/username/media/TV Shows/" --output "/home/hd*/username/renamedTVShows" -r --action move --db TheTVDB -non-strict --def excludeList=amc.txt "seriesFormat={n.replaceFirst(/^(?i)(The|A|An)\s(.+)/, /\$2, \$1/)} ({y}) /Season {s.pad(2)}/{n} - {s00e00} - {t} - {vf}.{vc.replace('x', 'h')}.{ac}.{af}{subt}"  --def clean=y --def subtitles=en --def "ut_label=TV" --conflict auto > filebotTV.txt
```

