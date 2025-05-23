# Introduction
This is an Arma 3 server container intended to be capable of being used as vanilla or modded.  

This readme should include all that you need to get the server up and running.  

Please feel free to open an issue with problems you're having, just remember to include logs or proof of the issue.

# Steps for Use
You'll need to do some specific steps due to the fact that downloading and updating Arma 3 server files with steamcmd requires a steam account that owns the game. Other sections under this one will also inform you on how to mount server files and the general directory of the container
## Caching user credentials
To download the server files, you'll need to login with a steam profile that owns Arma 3.  
To add steam user credentials to the container, follow these steps.
1. add `entrypoint: "/bin/bash"` to compose file
2. `sudo docker exec -it <container-name> bash`
3. `su arma` 
4. `steamcmd +login <steam-username>` 
5. login with password and steam guard if enabled
6. `quit` (from steamcmd)
7. `quit` (from exec session)
8. Remove the entrypoint line from the compose file
Your steam profile should be cached to allow the container to download updates for the server without having to repeat these steps  
It's slightly janky, but other options need a lot of work (which is a later issue if at all).

## Data Directory
All server files will be stored in the `./data` volume
- Server files will be within the `/data` directory
- Profile files will be located in `/data/.local/share/Arma 3 - Other Profiles/<username>` and the default profile will be located at `/data/.local/share/Arma 3`
    - To use files added to th Other Profiles folder, use the arma_user variable when starting the container

## Environment Variables

The following table has all current environment variables
| Name       | Desc                                                                                       | Default | Required |
| ---------- | ------------------------------------------------------------------------------------------ | ------- | -------- |
| steam_user | steam user for steamcmd to use for downloading sever files                                 | ""      | yes      |
| arma_user  | arma profile name to run server as if using files from "Arma 3 - Other Profiles" directory | ""      | no       |

## Ports Used
The following ports are used by the container

UDP ports 
- 2302
- 2303
- 2304
- 2305
- 2306
- 2344  

TCP ports
- 2344
- 2345

## Adding/Using Workshop Mods
To add mods to the server, create a file called `modlist.txt` and place it in the `/data` directory.  
To add mods with the modlist file, add the mods by taking the steamworkshop ID number and placing it in the text file each seperated by a line e.g.

```
2867537125
1638341685
1724884525
```
Planning to add the ability to add mods with an environment variable that has the workshop IDs comma serpeated in a single line in the case you don't want to make a text file with them.
# Docker Compose example
```
services:
  arma:
    image: huckmong/arma3-server:latest
    environment:
      steam_user: "steam"
    ports:
      - "2301:2301/udp"
      - "2302:2302/udp"
      - "2303:2303/udp"
      - "2304:2304/udp"
      - "2305:2305/udp"
      - "2306:2306/udp"
      - "2344:2344/udp"
      - "2345:2345"
    volumes:
      - ./data:/data
    restart: unless-stopped
    stdin_open: true
    tty: true
    #entrypoint: "/bin/bash"
```

# Common Issues

## Q: Some of the workshop items are failing to install/download
A: This seems to be a common issue when installing large workshop items as part of a bulk download. I recommend exec'ing into the container and then downloading the large mods individually with steamcmd.

## Q: My Mods aren't Loading!!!
A: This might be because you've left spaces in the modlist.txt document. Currently the container doesn't filter spaces from the lines and I intend to fix this eventually.  
EDIT: this has been fixed, the only thing that will ruin your modlist file now is if you add other numeric characters that aren't part of the workshop ID.
