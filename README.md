# Media Server

<!--
## Resources
- https://www.youtube.com/watch?v=LV3mcfqNgcQ
  - https://thomaswildetech.com/blog/2025/10/30/jellyfin---setting-up-the-entire-stack/
- https://www.youtube.com/watch?v=QfpZcXXGpVA
- https://www.youtube.com/watch?v=twJDyoj0tDc
- https://github.com/TRaSH-Guides/Guides
-->

## Introduction

This media stack contains the following applications

- Dockhand - Dockhand a utility that helps automate the updating and management of your docker containers
- Jellyfin - A free, open-source media server that organizes your movies and TV shows and streams them to all your devices
- Jellyseer - A request management tool that lets you browse a "Netflix-style" interface to request new content be added to your library
- Gluetun - A VPN client in a container to securely route traffic from other apps through a VPN
- qBittorrent - A lightweight, open-source BitTorrent client for downloading and managing torrents
- Prowlarr - An indexer manager that connects your download apps to various torrent and Usenet sites, syncing all those "search locations" in one central place.
- Radarr - A utility for locating Movies within your indexer and sending the request to your download application
- Sonarr - Same as Radarr but for TV Shows

## Server Setup

Requirements

- At least 20 GBs disk space needed
- 2 GBs of RAM

LXC Requirements

- Skip this section if you are installing on a VM or bare metal
- For LXC containers, pass-through the following devices:
  - `/dev/net/tun`
  - `/dev/dri/renderD128`
- If `id 1000` returns a user, use that user instead
- You must create a user for the docker containers to run as. Do not use the built-in root account.
  - Use `sudo useradd -s /usr/bin/bash -G sudo -m -u 1000 <username>` to create one if needed
    - This creates a user with a UID and GID of "1000", creates a home folder, sets their shell to bash, and adds them to the sudo group
  - Set a password for the new user `passwd <username>`
- Connect with SSH to your server with your user to continue

Create the following folder structure on the drive or network locations where you will be keeping your media and configs

- There are commands below the overview to quickly create the folders
```
data
├── config
│   ├── jellyfin-cache
│   └── jellyfin-config
├── downloads
└── media
    ├── movies
    └── shows
```

- First, navigate to the root folder of your storage location
```
cd /path/to/storage
```
- Create the folders
```
mkdir -p data/config/{jellyfin-cache,jellyfin-config}; \
mkdir -p data/downloads/; \
mkdir -p data/media/{movies,shows}
```

> [!NOTE]
> Ensure your user is the owner of data directory and it's sub folders.
>
> You may need to use `sudo chown <username>:<username> -R data` to take ownership of the files.
>
> Use `ls -la` to verify ownership.
>
> If the folders "jellyfin-cache" and "jellyfin-config" are not owned by your user Jellyfin will fail to start.

## Dockhand
Install docker
```
curl -fsSL https://get.docker.com | sudo sh
```

Install Dockhand to simplify stack management
```
sudo docker run -d \
  --name dockhand \
  --restart unless-stopped \
  -p 3000:3000 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v dockhand_data:/app/data \
  fnsys/dockhand:latest
```

Visit your server IP address at port `3000` to open the web interface

### Dockhand Environments
Click `Go to Settings` to configure your local environment

- Click `+ Add environment`
- Name it `local`
- Under "Public IP" add the IP address of the server. e.g. `192.168.0.126`
- Click `+ Add`
- Lastly click `Test all`

### Update Dockhand
While Dockhand supports updating images for other containers, it cannot update itself. These commands will stop, update, prune, and start Dockhand.
```
sudo docker stop dockhand; \
sudo docker pull fnsys/dockhand; \
sudo docker image prune -f -a; \
sudo docker start dockhand;
```

## Gluetun
Visit https://github.com/qdm12/gluetun-wiki/tree/main/setup/providers and select your VPN provider

- You'll need to login to your VPN's web interface to retrieve Open VPN credentials and location data
- Save this info in order to prepare your environment file in Dockhand

## Building the Stack

Pre-pull docker images to speed up building the stack
```
sudo docker pull ghcr.io/jellyfin/jellyfin; \
sudo docker pull qmcgaw/gluetun; \
sudo docker pull ghcr.io/linuxserver/qbittorrent; \
sudo docker pull ghcr.io/linuxserver/prowlarr; \
sudo docker pull ghcr.io/flaresolverr/flaresolverr; \
sudo docker pull ghcr.io/linuxserver/radarr; \
sudo docker pull ghcr.io/linuxserver/sonarr; \
sudo docker pull ghcr.io/linuxserver/bazarr; \
sudo docker pull ghcr.io/fallenbagel/jellyseerr; \
sudo docker pull ghcr.io/dockur/samba;
```

### Deploy the stack
Go to Stacks and select `+ Create`

- Give your stack a name at the top
  - _*Stack name must be lowercase, start with a letter or number, and contain only letters, numbers, hyphens, and underscores_
  - e.g. `media-server`
- Paste your compose file in the left side of the window
  - This is the contents of `mediaserver.yml`

Fill in your `.env` file

- To paste your `.env` file, you can either click `^ Load` button or change to edit mode by clicking the small document icon next to the words "Environment variables"
  - The file `.env.dist` contains a template of all the variables that need to be filled in
- "DATA_DIR" will be the path to your drive that media will be saved to. e.g. `/mnt/usb/data`
- Common Timezones:
  - America/New_York
  - America/Chicago
  - America/Denver
  - America/Phoenix
  - America/Los_Angeles
- Use `id <username>` to find your `uid` and `gid`, usually this is `1000` but yours may differ
- Under "Network Share Settings", you can change the name of the network share and specy the Username and Password to access it
- You must fill-in your VPN info into the Gluetun section or the container will restart over and over

Once you're ready, click the `Create & Start` button to deploy

- Run `sudo journalctl --unit docker -f` on the server to monitor for errors
- Startup time can vary from 1 to 3 minutes depending on hardware

## Jellyfin
Port: `8096`

Welcome to Jellyfin!

- Give your server a name. e.g. `Jellyfin` and click `Next`
- Set an admin Username and Password and click `Next`

Setup your media libraries

- Setup Movies
  - Click `Add Media Library`
  - For "Content type" select `Movies`
  - Set a "Display name" for the library
  - Click the `+` next to "Folders"
    - Navigate to `/data/media/movies` and click `OK`
  - Set "Preferred download language:" to `English`
  - Set "Country:" to `United States`
  - Click `OK`
- Setup Shows
  - Click `Add Media Library`
  - For "Content type" select `Shows`
  - Set a "Display name" for the library
  - Click the `+` next to "Folders"
    - Navigate to `/data/media/shows` and click `OK`
  - Click `Ok` when you've selected the folder
  - Set "Preferred download language:" to `English`
  - Set "Country:" to `United States`
  - Click `OK`
- Click `Next`
  - Verify "Language" and "Country/Region" then click `Next`
- Configure Remote Access
  - If you don't plan on the server being remotely accessible uncheck `Allow remote connections to this server.` otherwise click `Next`
- Click `Finish`

Set Default Audio

- User Profile > Playback > Audio Settings
- Set "Preferred audio language" to `English`
- Uncheck `Play default audio track regardless of language`
- Scroll to the bottom and click `Save`

Disable pagination in the web view

- User Profile > Display
- Scroll down to "Library page size:" and set the value to `0`
- Click `Save` at the bottom

## qBittorrent
Port: `8080`

After startup, open the log for qBittorrent and look for these lines. They will contain the login information.
```
The WebUI administrator username is: admin
The WebUI administrator password was not set. A temporary password is provided for this session: XXXXXXXXX
You should set your own password in program preferences.
```

After logging in, you should set your own password

- Tools > Options > WebUI
- Set a new Username and Password
- Don't forget to scroll to the bottom of the page and click `Save` every time you make a change

Set Default Download Location

- Tools > Options > Downloads
- Set "Default Save Path" to `/data/downloads`

Configure Bandwidth

- Tools > Options > Speed > Global Rate Limits
- Limit the upload speed
  - e.g. `5000` KiB/s or 10 percent of your ISP upload rate
- You can also optionally set alternative rates
  - This is intended to be a low speed mode which can switched on in the main download interface

Enable Torrent Queuing

- Tools > Options > BitTorrent > Torrent Queuing
- Make sure `Torrent Queuing` is checked
- Increase `Maximum active downloads` to `5` or another preferred value
- Check `Do not count slow torrents in these limits`
  - In order to use this properly be sure to also increase `Maximum active torrents`
  - e.g set `Maximum active torrents` to `10`
- If you uncheck `Torrent Queuing`, all torrents become active at the same time

Limit Seeding

- Tools > Options > BitTorrent > Seeding Limits
- Check `When ratio reaches` and set it to `0.00`
- Make sure `then` is set to `Stop torrent`
  - _*Radarr and Sonarr will remove the torrent after it's been imported_

Check Torrents On Completion

- Tools > Options > Advanced
- Uncheck `Confirm torrent recheck`
- Check `Recheck torrents on completion`

Use https://ipleak.net/ "Torrent Address detection" to verify VPN routing

## Prowlarr
Port: `9696`

Create login information

- Set "Authentication Method" to `Forms (Login Page)
- Set a value for Username and Password then click `Save`

Disable Telemetry

- Settings > General > Analytics
- Uncheck `Send Anonymous Usage Data`
- Click `Save Changes` at the top

<!-- This is required for new releases
Disable RSS

- Settings > Apps > Sync Profiles > Standard
- Uncheck `Enable RSS`
- Click `Save`
-->

Configure Minimum Seeders

- Settings > Apps > Sync Profiles > Standard
- Increase "Minimum Seeders" to `3` or greater

FlareSolverr

- Settings > Indexers > [+] > FlareSolverr
- Add a Tag. e.g. `flare` This is used later to indicate if an indexer requires a Cloudflare challenge
- Leave "Host" to it's default value of `http://localhost:8191/`
- Click `Save`

Add Indexers

- Indexers > Add New Indexer
- Find the indexer you want to add. Don't forget to include the `flare` tag if the site requires it.
  - This section is intentionally left vague, sorry.
  - If you are the author, check the Prowlarr-Indexer-List file
- You can use these filters to help find available public trackers
  - Protocol: `torrent`
  - Language: `en-US`
  - Privacy: `Public`
- After configuring an indexer, click `Test` and then `Save`

It's time to connect the Apps

## Radarr
Port: `7878`

Create login information

- Set "Authentication Method" to `Forms (Login Page)
- Set a value for Username and Password then click `Save`

Disable Telemetry

- Settings > General > Analytics
- Uncheck `Send Anonymous Usage Data`
- Click `Save Changes` at the top

Connect to Prowlarr

- Settings > General > Security
- Copy the `API Key`
- Go back to Prowlarr and Navigate to Settings > Apps > Applications > [+] > Radarr
- Paste the API Key
- Click `Test` and `Save`

Setup Media Folders

- Navigate back to Radarr
- Settings > Media Management
- Movie Naming
  - Enable `Rename Movies`
  - Colon Replacement: `Delete`
  - Standard Movie Format
    - Paste the string below:
    - `{Movie.CleanTitle}{.Release.Year}{.Edition.Tags}{.MediaInfo VideoCodec}{.Quality.Full}{-Release Group}`
- File Management
  - Enable `Unmonitor Deleted Movies`
- Root Folders
  - Click `Add Root Folder`
  - Navigate to `/data/media/movies/` and click `Ok`
- Click `Save Changes` at the top

Setup Download Client

- Settings > Download Clients > [+] > qBittorrent
- Fill in Username and Password for qBittorrent's WebUI
- Click `Test` and then `Save`

Setup a Quality Profile

- Settings > Custom Formats > [+]
- Name: `Standard Movie Size`
- Setup a Size condition
  - Condition > [+] > Size
  - Name: `Size`
  - Minimum Size: `.7`
  - Maximum Size: `4`
  - Check `Required` and click `Save`
- Optionally setup a codec condition (if you strictly only want an x264 or x265 release)
  - Condition > [+] > Release Title > Presets
  - Select x264 or x265 depending on you preference
  - Check `Required` and click `Save`
- Click `Save`
- Settings > Profiles
- For each of the following profiles: "HD-720p/1080p, HD-720p, HD-1080p", do the following:
  - Minimum Custom Format Score: `10`
  - Set "Standard Movie Size" score to `10`
  - Uncheck any Quality that includes "Remux"
  - Click `Save`

## Sonarr
Port: `8989`

Create login information

- Set "Authentication Method" to `Forms (Login Page)
- Set a value for Username and Password then click `Save`

Disable Telemetry

- Settings > General > Analytics
- Uncheck `Send Anonymous Usage Data`
- Click `Save Changes` at the top

Connect to Prowlarr

- Settings > General > Security
- Copy the `API Key`
- Go back to Prowlarr and Navigate to Settings > Apps > Applications > [+] > Sonarr
- Paste the API Key
- Click `Test` and `Save`

Setup Media Folders

- Navigate back to Sonarr
- Settings > Media Management
- Click `Show Advanced` in the top bar to show all option
- Episode Naming
  - Enable `Rename Episodes`
  - Colon Replacement: `Delete`
  - Standard Episode Format
    - Paste the string below:
    - `{Series.CleanTitleYear}.S{season:00}E{episode:00}.{Episode.CleanTitle}.{MediaInfo VideoCodec}.{Quality.Full}`
  - Series Folder Format
    - Set to `{Series TitleYear}`
- File Management
  - Enable `Unmonitor Deleted Episodes`
- Root Folders
  - Click `Add Root Folder`
  - Navigate to `/data/media/shows/` and click `Ok`
- Click `Save Changes` at the top

Setup Download Client

- Settings > Download Clients > [+] > qBittorrent
- Fill in Username and Password for qBittorrent's WebUI
- Click `Test` and then `Save`

Set Quality Settings

- Settings > Quality
- Adjust the presets to the chart below and then click `Save Changes` at the top

| Quality Contains | Min MB/m | Preferred MB/m | Max MB/m |
|------------------|----------|----------------|----------|
| SD/DVD/480p      | 4        | 15             | 20       |
| 720p             | 4        | 22             | 26       |
| 1080p            | 5        | 33             | 45       |

Prefer Season Packs

- Source: https://trash-guides.info/Sonarr/sonarr-collection-of-custom-formats/#season-pack
- Navigate to Settings > Custom Formats > [+]
- Click `Import` in the bottom left and paste the following
```
{
  "trash_id": "3bc5f395426614e155e585a2f056cdf1",
  "trash_scores": {
    "default": 10
  },
  "name": "Season Pack",
  "includeCustomFormatWhenRenaming": false,
  "specifications": [
    {
      "name": "Season Packs",
      "implementation": "ReleaseTypeSpecification",
      "negate": false,
      "required": false,
      "fields": {
        "value": 3
      }
    }
  ]
}
```
- Click `Save` and then navigate to Settings > Profiles
- For each of the following profiles: "HD-720p/1080p, HD-720p, HD-1080p", do the following:
  - Set "Season Pack" score to `10`
  - Click `Save`

## Jellyseer
Port: `5055`

Welcome to Jellyseerr

- Select `Configure Jellyfin`
- For "Jellyfin URL", just enter `jellyfin`
- Enter an email address followed by your Jellyfin credentials
- Click `Sign in`
- Click `Sync Libraries`
  - Toggle on `Movies` and `Shows`
- Click `Start Scan`
- Scroll to the bottom and click `Continue`

### Configure Services

Radarr

- Click `+ Add Radarr Server`
- Check `Default Server`
- Server Name: `Radarr`
- Hostname or IP Address: Enter your docker server IP Address
- API Key
  - In Radarr go to Settings > General > Security
  - Copy the `API Key`
  - Go back to Jellyseer and paste the Key
- Scroll to the bottom and click `Test`
- Quality Profile: `HD-1080p`
- Root Folder: `/date/media/movies`
- Click `Add Server`

Sonarr

- Click `+ Add Sonarr Server`
- Check `Default Server`
- Server Name: `Sonarr`
- Hostname or IP Address: Enter your docker server IP Address
- API Key
  - In Sonarr go to Settings > General > Security
  - Copy the `API Key`
  - Go back to Jellyseer and paste the Key
- Scroll to the bottom and click `Test`
- Quality Profile: `HD-720p`
- Root Folder: `/date/media/shows`
- Check `Season Folders`
- Click `Add Server`
- Click `Finish Setup`

## Bazarr
Port: `6767`

> [!NOTE]
> This section is a work-in-progress

Bazarr integrates with Sonarr and Radarr to download subtitles

- Create a free account at https://www.opensubtitles.com

https://www.youtube.com/watch?v=8vZ95HOdT-I&t=56s

https://wiki.bazarr.media/Getting-Started/Setup-Guide/

## SMB Network Share
To connect to the shared folder enter: `\\192.168.0.2\Data` in Windows Explorer, then enter your configured Username and Password.

> [!NOTE]
> Replace the example IP address above with that of your host.

For Macintosh use `smb://server/share`

<!-- future add-ons
Tdarr - Convert media and never worry about file sizes
-->
