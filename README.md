# How to Set Up a CS2 Server on Akash Network

This tutorial will guide you through the process of deploying a Counter-Strike 2 (CS2) server on the Akash Network using the Akash Console. Additionally, it will cover how to change the map and set up RCON for remote server management.

---

## Prerequisites
1. Akash Account: You need an Akash account with sufficient AKT tokens for deployment.
2. Akash Console: Access to the [Akash Console](https://console.akash.network).
3. CS2 Server Image: A Docker image for the CS2 server. You can use a pre-built image like `joedwards32/cs2` from Docker Hub.
4. Steam Game Server Login Token: Required for downloading CS2 server files. Obtain it from [Steam](https://steamcommunity.com/dev/managegameservers).

---

## Step 1: Deploy the CS2 Server on Akash Network

### 1.1 Create a Deployment Configuration
Create a deploy.yml file to define your CS2 server deployment. Below is an example configuration:
```
---
version: "2.0"
services:
  cs2:
    image: joedwards32/cs2
    expose:
      - port: 27015 # CS2 server listen port udp
        as: 27015
        proto: udp
        to:
          - global: true
      - port: 27020 # SourceTV/CSTV port
        as: 27020
        to:
          - global: true
      - port: 27050 # RCON port
        as: 27050
        to:
          - global: true
    env:
      - SRCDS_TOKEN=your_steam_token
      - CS2_IP=0.0.0.0
      - CS2_PORT=27015
      - CS2_LAN=0
      - CS2_RCON_PORT=27050
      - CS2_RCONPW=your_rcon_password
profiles:
  compute:
    cs2:
      resources:
        cpu:
          units: 4
        memory:
          size: 8Gi
        storage:
          - size: 60Gi
  placement:
    dcloud:
      pricing:
        cs2:
          denom: uakt
          amount: 1000
deployment:
  cs2:
    dcloud:
      profile: cs2
      count: 1
```
Replace the following placeholders:
- `your_steam_token`: Your Steam Game Server Login Token.
- `your_rcon_password`: A password for RCON access.

### 1.2 Deploy the Server
1. Log in to the [Akash Console](https://console.akash.network).
2. Navigate to the Deployments section and click Create Deployment.
3. Upload your `deploy.yml` file.
4. Review the deployment details and confirm.

Once the deployment is live, Akash will provide you with the server's IP address and ports.

---

## Step 2: Connect to the CS2 Server
1. Open Counter-Strike 2 on your PC.
2. Open the console and type: `connect IP:27015` with URI:PORT from Akash Console.

---

## Connect to RCON for Remote Management
RCON allows you to manage your CS2 server remotely. You can use [CS2 Web RCON Console](https://rcon.srcds.pro/) without installing any software.

1. Ensure the `CS2_RCONPW` environment variable is set in your `deploy.yml`.
2. Open [CS2 Web RCON Console](https://rcon.srcds.pro/) in your browser to connect to your server:
   - IP:PORT: URI and port you exposed for RCON from Akash Console.
   - RCON: The RCON password you set.
3. Change map with `map de_dust2` command.
4. Download and open map from Steam Workshop with `host_workshop_map 3379564935` command. Replace 3379564935 with ID from Steam Workshop.

---

## Environment Variables
Feel free to overwrite these environment variables, using `env` in your `deploy.yaml`: 

### Server Configuration

```
SRCDS_TOKEN=""              (Game Server Token from https://steamcommunity.com/dev/managegameservers)
CS2_SERVERNAME="changeme"   (Set the visible name for your private server)
CS2_CHEATS=0                (0 - disable cheats, 1 - enable cheats)
CS2_SERVER_HIBERNATE=0      (Put server in a low CPU state when there are no players. 
                             0 - hibernation disabled, 1 - hibernation enabled
                             n.b. hibernation has been observed to trigger server crashes)
CS2_IP=""                   (CS2 server listening IP address, 0.0.0.0 - all IP addresses on the local machine, empty - IP identified automatically)
CS2_PORT=27015              (CS2 server listen port tcp_udp)
CS2_RCON_PORT=""            (Optional, use a simple TCP proxy to have RCON listen on an alternative port.
                             Useful for services like AWS Fargate which do not support mixed protocol ports.)
CS2_LAN="0"                 (0 - LAN mode disabled, 1 - LAN Mode enabled)
CS2_RCONPW="changeme"       (RCON password)
CS2_PW=""                   (Optional, CS2 server password)
CS2_MAXPLAYERS=10           (Max players)
CS2_ADDITIONAL_ARGS=""      (Optional additional arguments to pass into cs2)
```

**Note:** When using `CS2_RCON_PORT` don't forget to map the port chosen with TCP protocol.

### Game Modes

```
CS2_GAMEALIAS=""            (Game type, e.g. casual, competitive, deathmatch.
                             See https://developer.valvesoftware.com/wiki/Counter-Strike_2/Dedicated_Servers)
CS2_GAMETYPE=0              (Used if CS2_GAMEALIAS not defined. See https://developer.valvesoftware.com/wiki/Counter-Strike_2/Dedicated_Servers)
CS2_GAMEMODE=1              (Used if CS2_GAMEALIAS not defined. See https://developer.valvesoftware.com/wiki/Counter-Strike_2/Dedicated_Servers)
CS2_MAPGROUP="mg_active"    (Map pool. Ignored if workshop maps are defined.)
CS2_STARTMAP="de_inferno"   (Start map. Ignored if workshop maps are defined.)
```

### Bots

```
CS2_BOT_DIFFICULTY=""       (0 - easy, 1 - normal, 2 - hard, 3 - expert)
CS2_BOT_QUOTA=""            (Number of bots)
CS2_BOT_QUOTA_MODE=""       (fill, competitive)
```

### CSTV/SourceTV

```
TV_ENABLE=0                 (0 - disable, 1 - enable)
TV_PORT=27020               (SourceTV/CSTV port to bind to)
TV_AUTORECORD=0             (Automatically record all games as CSTV demos: 0=off, 1=on)
TV_PW="changeme"            (CSTV password for clients)
TV_RELAY_PW="changeme"      (CSTV password for relay proxies)
TV_MAXRATE=0                (Max CSTV spectator bandwidth rate allowed, 0 == unlimited)
TV_DELAY=0                  (CSTV broadcast delay in seconds)
```

### Logs

```
CS2_LOG="on"                ('on'/'off')
CS2_LOG_MONEY=0             (Turns money logging on/off: 0=off, 1=on)
CS2_LOG_DETAIL=0            (Combat damage logging: 0=disabled, 1=enemy, 2=friendly, 3=all)
CS2_LOG_ITEMS=0             (Turns item logging on/off: 0=off, 1=on)
```

### Steam Workshop

Support for Steam Workshop is experimental!

```
CS2_HOST_WORKSHOP_MAP=""         (Steam Workshop Map ID to load on server start)   
CS2_HOST_WORKSHOP_COLLECTION=""  (Steam Workshop Collection ID to download)
```

Environment in deploy.yaml: `CS2_HOST_WORKSHOP_MAP=3379564935` will start server with map from Steam Workshop with ID [3379564935](https://steamcommunity.com/sharedfiles/filedetails/?id=3379564935).

If a Workshop Collection is set, maps can be selected via rcon. E.g:

```
ds_workshop_listmaps
ds_workshop_changelevel $map_name
```

---

## Monitor and Maintain Your Server
- Use the [Akash Console](https://console.akash.network) to monitor your deployment and check logs.
- To update the server, modify your `deploy.yml` and redeploy.
- To stop the server, delete the deployment from the Akash Console.

---

## Troubleshooting
- Server Not Starting: Check the logs in the Akash Console for errors. Ensure your Steam API key is valid.
- Connection Issues: Verify the server IP and port. Ensure the firewall allows traffic on the specified ports.
- RCON Not Working: Double-check the RCON password and ensure the RCON port is exposed.

---

## Note for Zealy
SDL succesfully tested on providers provider.bozolinux.com and provider.akash-palmito.org.

---

Congratulations! You now have a fully functional CS2 server running on the Akash Network. Enjoy your game!
