[![Docker][docker-image]][docker-url]

[docker-image]: https://img.shields.io/docker/pulls/salvoxia/immich-folder-album-creator.svg
[docker-url]: https://hub.docker.com/r/salvoxia/immich-folder-album-creator/

# Immich Folder Album Creator

This is a python script designed to automatically create albums in [Immich](https://immich.app/) from a folder structure mounted into the Immich container.
This is useful for automatically creating and populating albums for external libraries.
Using the provided docker image, the script can simply be added to the Immich compose stack and run along the rest of Immich's containers.

__Current compatibility:__ Immich v1.106.1 - v1.112.x

## Disclaimer
This script is mostly based on the following original script: [REDVM/immich_auto_album.py](https://gist.github.com/REDVM/d8b3830b2802db881f5b59033cf35702)

# Table of Contents
1. [Usage (Bare Python Script)](#bare-python-script)
2. [Usage (Docker)](#docker)
3. [Choosing the correct `root_path`](#choosing-the-correct-root_path)
4. [How It Works (with Examples)](#how-it-works)
5. [Automatic Album Sharing](#automatic-album-sharing)
6. [Cleaning Up Albums](#cleaning-up-albums)
7. [Dealing with External Library Changes](#dealing-with-external-library-changes)

## Usage
### Bare Python Script
1. Download the script and its requirements
    ```bash
    curl https://raw.githubusercontent.com/Salvoxia/immich-folder-album-creator/main/immich_auto_album.py -o immich_auto_album.py
    curl https://raw.githubusercontent.com/Salvoxia/immich-folder-album-creator/main/requirements.txt -o requirements.txt
    ```
2. Install requirements
    ```bash
    pip3 install -r requirements.txt
    ```
3. Run the script
    ```
    usage: immich_auto_album.py [-h] [-r ROOT_PATH] [-u] [-a ALBUM_LEVELS] [-s ALBUM_SEPARATOR] [-c CHUNK_SIZE] [-C FETCH_CHUNK_SIZE] [-l {CRITICAL,ERROR,WARNING,INFO,DEBUG}] [-k] [-i IGNORE] [-m {CREATE,CLEANUP,DELETE_ALL}] [-d] [-x SHARE_WITH] [-o {viewer,editor}]
                            [-S {0,1,2}] [-O {False,asc,desc}]
                            root_path api_url api_key

    Create Immich Albums from an external library path based on the top level folders

    positional arguments:
      root_path             The external libarary's root path in Immich
      api_url               The root API URL of immich, e.g. https://immich.mydomain.com/api/
      api_key               The Immich API Key to use

    options:
      -h, --help            show this help message and exit
      -r ROOT_PATH, --root-path ROOT_PATH
                            Additional external libarary root path in Immich; May be specified multiple times for multiple import paths or external libraries. (default: None)
      -u, --unattended      Do not ask for user confirmation after identifying albums. Set this flag to run script as a cronjob. (default: False)
      -a ALBUM_LEVELS, --album-levels ALBUM_LEVELS
                            Number of sub-folders or range of sub-folder levels below the root path used for album name creation. Positive numbers start from top of the folder structure, negative numbers from the bottom. Cannot be 0. If a range should be set, the
                            start level and end level must be separated by a comma like '<startLevel>,<endLevel>'. If negative levels are used in a range, <startLevel> must be less than or equal to <endLevel>. (default: 1)
      -s ALBUM_SEPARATOR, --album-separator ALBUM_SEPARATOR
                            Separator string to use for compound album names created from nested folders. Only effective if -a is set to a value > 1 (default: )
      -c CHUNK_SIZE, --chunk-size CHUNK_SIZE
                            Maximum number of assets to add to an album with a single API call (default: 2000)
      -C FETCH_CHUNK_SIZE, --fetch-chunk-size FETCH_CHUNK_SIZE
                            Maximum number of assets to fetch with a single API call (default: 5000)
      -l {CRITICAL,ERROR,WARNING,INFO,DEBUG}, --log-level {CRITICAL,ERROR,WARNING,INFO,DEBUG}
                            Log level to use (default: INFO)
      -k, --insecure        Set to true to ignore SSL verification (default: False)
      -i IGNORE, --ignore IGNORE
                            A string containing a list of folders, sub-folder sequences or file names separated by ':' that will be ignored. (default: )
      -m {CREATE,CLEANUP,DELETE_ALL}, --mode {CREATE,CLEANUP,DELETE_ALL}
                            Mode for the script to run with. CREATE = Create albums based on folder names and provided arguments; CLEANUP = Create album nmaes based on current images and script arguments, but delete albums if they exist; DELETE_ALL = Delete all
                            albums. If the mode is anything but CREATE, --unattended does not have any effect. Only performs deletion if -d/--delete-confirm option is set, otherwise only performs a dry-run. (default: CREATE)
      -d, --delete-confirm  Confirm deletion of albums when running in mode CLEANUP or DELETE_ALL. If this flag is not set, these modes will perform a dry run only. Has no effect in mode CREATE (default: False)
      -x SHARE_WITH, --share-with SHARE_WITH
                            A user name (or email address of an existing user) to share newly created albums with. Sharing only happens if the album was actually created, not if new assets were added to an existing album. If the the share role should be specified by
                            user, the format <userName>=<shareRole> must be used, where <shareRole> must be one of 'viewer' or 'editor'. May be specified multiple times to share albums with more than one user. (default: None)
      -o {viewer,editor}, --share-role {viewer,editor}
                            The default share role for users newly created albums are shared with. Only effective if --share-with is specified at least once and the share role is not specified within --share-with. (default: viewer)
      -S {0,1,2}, --sync-mode {0,1,2}
                            Synchronization mode to use. Synchronization mode helps synchronizing changes in external libraries structures to Immich after albums have already been created. Possible Modes: 0 = do nothing; 1 = Delete any empty albums; 2 = Trigger
                            offline asset removal (REQUIRES API KEY OF AN ADMIN USER!) (default: 0)
      -O {False,asc,desc}, --album-order {False,asc,desc}
                            Set sorting order for newly created albums to newest or oldest file first, Immich defaults to newest file first (default: False)
    ```

__Plain example without optional arguments:__
```bash
python3 ./immich_auto_album.py /path/to/external/lib https://immich.mydomain.com/api thisIsMyApiKeyCopiedFromImmichWebGui
```

### Docker

A Docker image is provided to be used as a runtime environment. It can be used to either run the script manually, or via cronjob by providing a crontab expression to the container. The container can then be added to the Immich compose stack directly.

#### Environment Variables
The environment variables are analoguous to the script's command line arguments.

| Environment varible   |  Mandatory? | Description   |
| :------------------- | :----------- | :------------ |
| ROOT_PATH            | yes | A single or a comma separated list of import paths for external libraries in Immich. <br>Refer to [Choosing the correct `root_path`](#choosing-the-correct-root_path).|
| API_URL            | yes | The root API URL of immich, e.g. https://immich.mydomain.com/api/ |
| API_KEY            | yes | The Immich API Key to use  
| CRON_EXPRESSION    | yes | A [crontab-style expression](https://crontab.guru/) (e.g. `0 * * * *`) to perform album creation on a schedule (e.g. every hour). |
| ALBUM_LEVELS       | no | Number of sub-folders or range of sub-folder levels below the root path used for album name creation. Positive numbers start from top of the folder structure, negative numbers from the bottom. Cannot be `0`. If a range should be set, the start level and end level must be separated by a comma. <br>Refer to [How it works](#how-it-works) for a detailed explanation and examples. |
| ALBUM_SEPARATOR    | no | Separator string to use for compound album names created from nested folders. Only effective if `-a` is set to a value `> 1`(default: "` `") |
| CHUNK_SIZE         | no | Maximum number of assets to add to an album with a single API call (default: `2000`)  |
| FETCH_CHUNK_SIZE   | no | Maximum number of assets to fetch with a single API call (default: `5000`)            |
| LOG_LEVEL          | no | Log level to use (default: INFO), allowed values: `CRITICAL`,`ERROR`,`WARNING`,`INFO`,`DEBUG` |
| INSECURE           | no | Set to `true` to disable SSL verification for the Immich API server, useful for self-signed certificates (default: `false`), allowed values: `true`, `false` |
| INSECURE           | no | A string containing a list of folders, sub-folder sequences or file names separated by '`:`' that will be ignored. |
| MODE               | no | Mode for the script to run with. <br> __`CREATE`__ = Create albums based on folder names and provided arguments<br>__`CLEANUP`__ = Create album nmaes based on current images and script arguments, but delete albums if they exist <br> __`DELETE_ALL`__ = Delete all albums. <br> If the mode is anything but `CREATE`, `--unattended` does not have any effect. <br> (default: `CREATE`). <br>Refer to [Cleaning Up Albums](#cleaning-up-albums). |
| DELETE_CONFIRM     | no | Confirm deletion of albums when running in mode `CLEANUP` or `DELETE_ALL`. If this flag is not set, these modes will perform a dry run only. Has no effect in mode `CREATE` (default: `False`). <br>Refer to [Cleaning Up Albums](#cleaning-up-albums).|
| SHARE_WITH     | no | A single or a colon (`:`) separated list of existing user names (or email addresses of existing users) to share newly created albums with. If the the share role should be specified by user, the format <userName>=<shareRole> must be used, where <shareRole> must be one of `viewer` or `editor`. May be specified multiple times to share albums with more than one user. (default: None) Sharing only happens if an album is actually created, not if new assets are added to it.  <br>Refer to [Automatic Album Sharing](#automatic-album-sharing).|
| SHARE_ROLE     | no | The role for users newly created albums are shared with. Only effective if `SHARE_WITH` is not empty and no explicit share role was specified for at least one user. (default: viewer), allowed values: `viewer`, `editor` |
| SYNC_MODE     | no | Synchronization mode to use. Synchronization mode helps synchronizing changes in external libraries structures to Immich after albums have already been created. Possible Modes: <br>`0` = do nothing<br>`1` = Delete any empty albums<br>`2` = Trigger offline asset removal (REQUIRES API KEY OF AN ADMIN USER!)<br>(default: `0`)<br>Refer to [Dealing with External Library Changes](#dealing-with-external-library-changes). |
| ALBUM_ORDER     | no | Set sorting order for newly created albums to newest (`desc`) or oldest (`asc`) file first, Immich defaults to newest file first, allowed values: `asc`, `desc` |

#### Run the container with Docker

To perform a manually triggered __dry run__ (only list albums that __would__ be created), use the following command:
```bash
docker run -e API_URL="https://immich.mydomain.com/api/" -e API_KEY="xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" -e ROOT_PATH="/external_libs/photos" salvoxia/immich-folder-album-creator:latest /script/immich_auto_album.sh
```
To actually create albums after performing a dry run, use the following command (setting the `UNATTENDED` environment variable):
```bash
docker run -e UNATTENDED="1" -e API_URL="https://immich.mydomain.com/api/" -e API_KEY="xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" -e ROOT_PATH="/external_libs/photos" salvoxia/immich-folder-album-creator:latest /script/immich_auto_album.sh
```

To set up the container to periodically run the script, give it a name, pass the TZ variable and a valid crontab expression as environment variable. This example runs the script every hour:
```bash
docker run --name immich-folder-album-creator -e TZ="Europe/Berlin" -e CRON_EXPRESSION="0 * * * *" -e API_URL="https://immich.mydomain.com/api/" -e API_KEY="xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" -e ROOT_PATH="/external_libs/photos" salvoxia/immich-folder-album-creator:latest
```

If your external library uses multiple import paths or you have set up multiple external libraries, you can pass multiple paths in `ROOT_PATH` by setting it to a comma separated list of paths:  
```bash
docker run --name immich-folder-album-creator -e TZ="Europe/Berlin" -e CRON_EXPRESSION="0 * * * *" -e API_URL="https://immich.mydomain.com/api/" -e API_KEY="xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" -e ROOT_PATH="/external_libs/photos,/external_libs/more_photos" salvoxia/immich-folder-album-creator:latest
```

### Run the container with Docker-Compose

Adding the container to Immich's `docker-compose.yml` file:
```yml
#
# WARNING: Make sure to use the docker-compose.yml of the current release:
#
# https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
#
# The compose file on main may not be compatible with the latest release.
#

name: immich

services:
  immich-server:
    container_name: immich_server
    volumes:
     - /path/to/my/photos:/external_libs/photos
  ...
  immich-folder-album-creator:
    container_name: immich_folder_album_creator
    image: salvoxia/immich-folder-album-creator:latest
    restart: unless-stopped
    environment:
      API_URL: http://immich_server:3001/api
      API_KEY: xxxxxxxxxxxxxxxxx
      ROOT_PATH: /external_libs/photos
      CRON_EXPRESSION: "0 * * * *"
      TZ: Europe/Berlin

```

## Choosing the correct `root_path`
The root path  `/path/to/external/lib/` is the path you have mounted your external library into the Immich container.  
If you are following [Immich's External library Documentation](https://immich.app/docs/guides/external-library), you are using an environment variable called `${EXTERNAL_PATH}` which is mounted to `/usr/src/app/external` in the Immich container. Your `root_path` to pass to the script is `/usr/src/app/external`.

## How it works

The script utilizies [Immich's REST API](https://immich.app/docs/api/) to query all images indexed by Immich, extract the folder for all images that are in the top level of any provided `root_path`, then creates albums with the names of these folders (if not yet exists) and adds the images to the correct albums.

The following arguments influence what albums are created:  
`root_path`, `--album-levels` and `--album-separator`  

  - `root_path` is the base path where images are looked for. Multiple root paths can be specified by adding the `-r` argument as many times as necessary. Only images within that base path will be considered for album creation.
  - `--album-levels` controls how many levels of nested folders are considered when creating albums. The default is `1`. For examples see below.
  - `--album-separator` sets the separator used for concatenating nested folder names to create an album name. It is a blank by default.

__Examples:__  
Suppose you provide an external library to Immich under the path `/external_libs/photos`.
The folder structure of `photos` might look like this:

```
/external_libs/photos/2020
/external_libs/photos/2020/02 Feb
/external_libs/photos/2020/02 Feb/Vacation
/external_libs/photos/2020/08 Aug/Vacation
/external_libs/photos/Birthdays/John
/external_libs/photos/Birthdays/Jane
/external_libs/photos/Skiing 2023
```

Albums created for `root_path = /external_libs/photos` (`--album-levels` is implicitly set to `1`):
 - `2020` (containing all images from `2020` and all sub-folders)
 - `Birthdays` (containing all images from Birthdays itself as well as `John` and `Jane`)
 - `Skiing 2023`

 Albums created for `root_path = /external_libs/photos` (`--album-levels` is implicitly set to `1`) and `--ignore "Vacation"`:
 - `2020` (containing all images from `2020`, `2020/02 Feb` and `2020/08 Aug`, but __NOT__ `2020/02 Feb/Vacation` or `2020/08 Aug/Vacation`)
 - `Birthdays` (containing all images from Birthdays itself as well as `John` and `Jane`)
 - `Skiing 2023`

Albums created for `root_path = /external_libs/photos/Birthdays`:
 - `John`
 - `Jane`

 Albums created for `root_path = /external_libs/photos` and `--album-levels = 2`:
 - `2020` (containing all images from `2020` itself, if any)
 - `2020 02 Feb` (containing all images from `2020/02 Feb` itself, `2020/02 Feb/Vacation` and `2020/02 Aug/Vacation`)
 - `Birthdays John` (containing all imags from `Birthdays/John`)
 - `Birthdays Jane` (containing all imags from `Birthdays/John`)
 - `Skiing 2023`

 Albums created for `root_path = /external_libs/photos`, `--album-levels = 3` and `--album-separator " - "` :
 - `2020` (containing all images from `2020` itself, if any)
 - `2020 - 02 Feb` (containing all images from `02 Feb` itself, if any)
 - `2020 - 02 Feb - Vacation` (containing all imags from `2020/02 Feb/Vacation`)
 - `2020 - 08 Aug - Vacation` (containing all imags from `2020/02 Aug/Vacation`)
 - `Birthdays - John`
 - `Birthdays - Jane`
 - `Skiing 2023`

  Albums created for `root_path = /external_libs/photos`, `--album-levels = -1` and `--album-separator " - "` :
 - `2020` (containing all images from `2020` itself, if any)
 - `02 Feb` (containing all images from `2020/02 Feb` itself, if any)
 - `Vacation` (containing all images from `2020/02 Feb/Vacation` AND `2020/08 Aug/Vacation`)
 - `John` (containing all imags from `Birthdays/John`)
 - `Jane` (containing all imags from `Birthdays/Jane`)
 - `Skiing 2023`
 
 ### Album Level Ranges

 It is possible to specify not just a number for `--album-levels`, but a range from level x to level y in the folder structure that should make up an album's name:  
 `--album-levels="2,3"`  
 The range is applied to the folder structure beneath `root_path` from the top for positive levels and from the bottom for negative levels.
 Suppose the following folder structure for an external library with the script's `root_path` set to `/external_libs/photos`:
 ```
/external_libs/photos/2020/2020 02 Feb/Vacation
/external_libs/photos/2020/2020 08 Aug/Vacation
```
 - `--album-levels="2,3"` will create albums (for this folder structure, this is equal to `--album-levels="-2"`)
    - `2020 02 Feb Facation`
    - `2020 08 Aug Vacation`
  - `--album-levels="2,2"` will create albums (for this folder structure, this is equal to `--album-levels="-2,-2"`)
    - `2020 02 Feb`
    - `2020 08 Aug`

⚠️ __Important:__ When passing negative ranges as album levels, you __must__ pass the argument in the form `--album-levels="-2,-2"`. Emphasis is on the equals sign `=` separating the option from the value. Otherwise, you might get an error `argument -a/--album-levels: expected one argument`!

⚠️ __Attention:__ Note that with negative `album-levels` or album level ranges, images from different parent folders will be mixed in the same album if they reside in sub-folders with the same name (see `Vacation` in example above).

Since Immich does not support real nested albums ([yet?](https://github.com/immich-app/immich/discussions/2073)), neither does this script.

## Automatic Album Sharing

The scripts support sharing newly created albums with a list of existing users. The sharing role (`viewer` or `editor`) can be specified for all users at once or individually per user.

### Album Sharing Examples (Bare Python Script)
Two arguments control this feature:
  - `-o / --share-role`: The default role for users an album is shared with. Allowed values are `viewer` or `editor`. This argument is optional and defaults to `viewer`.
  - `-x / --share-with`: Specify once per user to share with. The value should be either the user name or the user's email address as specified in Immich. If the user name is used and it contains blanks ` `, it must be wrapped in double quotes `"`. To override the default share role and specify a role explicitly for this user, the format `<userName>=<shareRole>` must be used (refer to examples below).

To share new albums with users `User A` and `User B` as `viewer`, use the following call:
```bash
python3 ./immich_auto_album.py --share-with "User A" --share-with "User B" /path/to/external/lib https://immich.mydomain.com/api thisIsMyApiKeyCopiedFromImmichWebGui
```

To share new albums with users `User A` and `User B` as `editor`, use the following call:
```bash
python3 ./immich_auto_album.py --share-with "User A" --share-with "User B" --share-role "editor" /path/to/external/lib https://immich.mydomain.com/api thisIsMyApiKeyCopiedFromImmichWebGui
```

To share new albums with users `User A` and a user with mail address `userB@mydomain.com`, but `User A` should be an editor, use the following call:
```bash
python3 ./immich_auto_album.py --share-with "User A=editor" --share-with "userB@mydomain.com" /path/to/external/lib https://immich.mydomain.com/api thisIs
```


### Album Sharing Examples (Docker)
Two environment variables control this feature:
  - `SHARE_ROLE`: The default role for users an album is shared with. Allowed values are `viewer` or `editor`. This argument is optional and defaults to `viewer`.
  - `SHARE_WITH`: A colon `:` separated list of either names or email addresses (or a mix) of existing users. To override the default share role and specify a role explicitly for each user, the format `<userName>=<shareRole>` must be used (refer to examples below).

To share new albums with users `User A` and `User B` as `viewer`, use the following call:
```bash
docker run -e SHARE_WITH="User A:User B" -e UNATTENDED="1" -e API_URL="https://immich.mydomain.com/api/" -e API_KEY="xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" -e ROOT_PATH="/external_libs/photos" salvoxia/immich-folder-album-creator:latest /script/immich_auto_album.sh
```

To share new albums with users `User A` and `User B` as `editor`, use the following call:
```bash
docker run -e SHARE_WITH="User A:User B" -e SHARE_ROLE="editor" -e UNATTENDED="1" -e API_URL="https://immich.mydomain.com/api/" -e API_KEY="xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" -e ROOT_PATH="/external_libs/photos" salvoxia/immich-folder-album-creator:latest /script/immich_auto_album.sh
```

To share new albums with users `User A` and a user with mail address `userB@mydomain.com`, but `User A` should be an editor, use the following call:
```bash
docker run -e SHARE_WITH="User A=editor:userB@mydomain.com" -e UNATTENDED="1" -e API_URL="https://immich.mydomain.com/api/" -e API_KEY="xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" -e ROOT_PATH="/external_libs/photos" salvoxia/immich-folder-album-creator:latest /script/immich_auto_album.sh
```


## Cleaning Up Albums

The script supports differnt run modes (option `-m`/`--mode` or env variable `MODE` for Docker). The default mode is `CREATE`, which is used to create albums.
The other two modes are `CLEANUP` and `DELETE_ALL`:
  - `CLEANUP`: The script will generate album names using the script's arguments and the assets found in Immich, but instead of creating the albums, it will delete them (if they exist). This is useful if a large number of albums was created with no/the wrong `--album-separator` or `--album-levels` settings.
  - `DELETE_ALL`: ⚠️ As the name suggests, this mode blindly deletes ALL albums from Immich. Use with caution!

To prevent accidental deletions, setting the mode to `CLEANUP` or `DELETE_ALL` alone will not actually delete any albums, but only perform a dry run. The dry run prints a list of albums that the script __would__ delete.  
To actually delete albums, the option `-d/--delete-confirm` (or env variable `DELETE_CONFIRM` for Docker) must be set.

__WARNING ⚠️__  
Deleting albums cannot be undone! The only option is to let the script run again and create new albums base on the passed arguments and current assets in Immich.

## Dealing with External Library Changes

Due to their nature, external libraries may be changed by the user without Immich having any say in it.  
Two examples of this are the user deleting or renaming folders in their external libraries after the script has created albums, or the user moving single files from one folder to another. The script would create new albums from renamed folders or add images to their new album after they have been moved.  
Immich itself deals with this by marking images/videos it no longer sees in their original location as "offline". This can lead to albums being completely populated by "offline" files only (if the folder was renamed or deleted) while they exist normally in a new album or with single images being offline in one album, while existing normally in their new albums.  
There is an administrative option to trigger deletion of "offline assets" from libraries in the Immich Admin Interface. However, in the first example this might leave behind empty albums if the album only contained offline files.

To clean up this mess this script offers an option called `Sync Mode`. It is an optional argument / environment variable that may have values from 0 to 2.
The following behaviors wil be triggered by the different values:
  - `0`: No syncing (default)
  - `1`: Delete all empty albums at the end of a run
  - `2`: Trigger offline asset removal for all libraires (⚠️ This function requires the API key of an Admin User!)

The complete syncing process consists of two stages:
1. Trigger Offline Asset Removal
This will start a background job for each library to find and delete files from Immich's database that are marked as 'offline'. It is asynchronous and might take some time (depending on the size of the databases and the numver of offline files).
Offline Asset Removal can only be triggered by Administrators, so the API Key used for this sync mode must be one of an Administrator. This might not be the same you use for creating albums!

2. Delete Empty Albums
Step 1 might leave behind empty albums for a different user. The script is capable of removing empty albums at the end of a run to clean up after Offline Asset Removal.
⚠️ It is __not__ possible for the script to distinguish between an album that was left behind empty after Offline Asset Removal and a manually created album with no images added to it! All empty albums of that user will be deleted!

It is up to you whether you want to use the full capabilities Sync Mode offers, parts of it or none.  
An example for the Immich `docker-compose.yml` stack when using full Sync Mode might look like this:
```yml
#
# WARNING: Make sure to use the docker-compose.yml of the current release:
#
# https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
#
# The compose file on main may not be compatible with the latest release.
#

name: immich

services:
  immich-server:
    container_name: immich_server
    volumes:
     - /path/to/my/photos:/external_libs/photos
  ...
  immich-folder-album-creator-offline-asset-removal:
    container_name: immich_folder_album_creator
    image: salvoxia/immich-folder-album-creator:latest
    restart: unless-stopped
    environment:
      API_URL: http://immich_server:3001/api
      # Admin User API Key
      API_KEY: yyyyyyyyyyyyyyyyyy
      ROOT_PATH: /thisIsADummyPathThatDoesNotExist
      CRON_EXPRESSION: "0 * * * *"
      TZ: Europe/Berlin
      # Regularly trigger offline asset removal
      SYNC_MODE: "2"
  immich-folder-album-creator:
    container_name: immich_folder_album_creator
    image: salvoxia/immich-folder-album-creator:latest
    restart: unless-stopped
    environment:
      API_URL: http://immich_server:3001/api
      API_KEY: xxxxxxxxxxxxxxxxx
      ROOT_PATH: /external_libs/photos
      # Run 10 minutes past the hour to give Offline Asset Removal time to work
      CRON_EXPRESSION: "10 * * * *"
      TZ: Europe/Berlin
      # Delete empty albums after each run
      SYNC_MODE: "1"
```

The `immich-folder-album-creator-offline-asset-removal` service has the one and only purpose of triggering Offline Asset Removal, since we do not want to create albums for the Admin User. Thus, a dummy `ROOT_PATH` is passed so that the script will never find any assets to create albums from. `SYNC_MODE` is set to `2` to regularly trigger Offline Asset Removal.  
The `immich-folder-album-creator` service is the normal service for creating albums from an external library. The cron job triggers 10 minutes past the hour, whereas Offline Asset Removal is trigger at the full hour. This gives the job 10 minutes of time to run before any empty albums are deleted at the end of the run due to `SYNC_MODE` being set to `1`.
