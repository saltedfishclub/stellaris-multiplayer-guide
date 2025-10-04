# Stellaris Multiplayer Guide

Many players have suffered from the built-in multiplayer feature of Stellaris. This guide will help you bypass Steam and connect with your friends over a VPN, offering a more tunable networking setup that can lead to lower latency and fewer desynchronizations.

> [!NOTE]
> You must own a legitimate copy of Stellaris.  
> Although we use tools from the piracy community, we strongly recommend using clean game files from Steam. Otherwise, we cannot guarantee that this guide will work.  
> Specifically, this guide is based on Stellaris version 4.1.1 hotfix.

## Instructions

### 1. Ensure Connectivity Between Players (Usually via VPN)

We recommend using [Tailscale](https://tailscale.com). Simply install it on each computer and log in with the same account (or join the same network). Once connected, you should be able to reach other players.

How to check connectivity (using Tailscale as an example):

1. Press `Win + X`, then press `I` to open the terminal.
2. Type `tailscale status`. You should see a list of machines that have joined your VPN, even if some are offline.  
   Example output:
   ```
   100.16.5.0    ____                 deno.tail06a5b.ts.net windows -
   100.1.1.4     _______              tagged-devices linux   offline
   100.15.23.12  __________________   tagged-devices windows -
   ```
3. The first column shows the IP address. You can only ping machines that are online (check the last column).
   ```
   C:\Users\...> ping 100.16.5.0
   ```
   If you don’t see messages like `timed out`, the connection is successful. Note that latency may be high initially, as Tailscale may need time to establish a direct (P2P) connection.

---

### 2. Apply the Patch

Visit the [releases page](https://github.com/saltedfishclub/stellaris-multiplayer-guide/releases) and download the latest `steam_api64.dll`. This is a modified version of the Goldberg Emulator tailored for Stellaris.

> If you don’t trust the precompiled binary, you can [compile it yourself](#compiling-goldberg).

Once you have the DLL:

- Place it in your Stellaris installation folder and **overwrite** the original file.
- Create the following files:

> [!NOTE]  
> In the instructions below, `.` refers to the Stellaris installation folder.

- Ensure `steam_appid.txt` exists and contains:
  ```
  281990
  ```
- Create a folder: `./steam_settings`
- Create `./steam_settings/listen_port.txt` with content:
  ```
  47584
  ```

These steps must be done **on every player’s machine**, regardless of whether they are hosting or joining.

---

### 3. Host Setup (Only for the Host)

If you're using Tailscale, open the terminal and type:

```
tailscale ip
```

The first line is your **VPN IP address**. Share this IP with all other players.

Each player must create `./steam_settings/custom_broadcasts.txt` with the following content:

```
<host_vpn_ip>:47584
```

For example:
```
114.514.19.19:47584
```

> If you changed the port in `listen_port.txt`, update the port number accordingly.

Your final folder structure should look like this:

```
.
|-- steam_api64.dll
|-- steam_appid.txt
`-- steam_settings
    |-- custom_broadcasts.txt
    `-- listen_port.txt
```

---

### 4. Launch the Game

Double-click `stellaris.exe` or launch the game via the launcher. If you can access the multiplayer lobby without crashes, and other players can see your lobby, the setup is working.

> If the game crashes when entering the multiplayer lobby, you are **not** using our modified version of Goldberg.

If you still experience lag and it’s no better than Steam’s multiplayer, the issue is likely your network. You may want to try other VPN software or even FRP.

> [!NOTE]
> If you have a public IPv4 address, Tailscale should automatically use it. In that case, there’s no need to manually configure public IP settings, which could expose you to security risks.

---

## Compiling Goldberg

For unknown reasons, the original **Goldberg Emulator** crashes the game when entering the lobby list, even when using the `+connect_lobby` launch parameter.

You need to patch this: 
https://gitlab.com/Mr_Goldberg/goldberg_emulator/-/blob/master/dll/steam_matchmaking_servers.cpp#L475

```patch
for (auto &r : requests) {
-    if (r.cancelled || r.completed) continue;
+    continue;
    r.gameservers_filtered.clear();
    ...
}
```

This change does not affect the lobby list functionality and completely resolves the crash issue. You can apply this patch and compile the emulator yourself if desired.

---

## License

This guide is licensed under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/).