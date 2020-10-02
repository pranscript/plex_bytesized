# Nzbget

- The first thing you need is a Usenet server.
- We will be using Nzbget only for downloading Movies
- Only two things needs to be changed in the Nzbget settings.
- Got to settings and select News-servers and input the information about your news server. 

1) Usenet Server settings

![GitHub Logo](../images/NZB1.jpg)

- Choose any name
- Level 0 is necessary if you have only 1 server.
- Put Group as 0
- Host is the URL if will be connecting to! This information will be available to you in the Usenet dashboard.

![GitHub Logo](../images/NZB2.jpg)

- Port 563 is used as SSL port. You must have two ports given to you. Use SSL port here.
- Put your username of Usenet server.
- Password of Usenet server.
- Encryption as "Yes" id you put SSL port. "No" if you put normal port.
- Connections provided to you by your provider. Remember Sonarr and Radarr will use the same connection so divide between them.

2) Remove Filebot

- Go to extension scripts and remove filebot.py.
- This is done as we will let Radarr do the renaming stuffs. 

Scroll down, save the changes and and reload. Nzbget Done!!