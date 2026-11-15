# Tailscale on pfSense

Install the Tailscale package on pfSense so your phone or laptop can reach your home network from anywhere, with no port forwarding.

▶️ Video: https://youtu.be/HDFw4EVDZj8

The addresses below are the ones from the video. Change them to yours:

| | Address | What it is |
|---|---|---|
| pfSense | `10.37.0.1` | the pfSense box (web page on port `65000` in the video) |
| Network to share | `10.37.0.0/20` | the server VLAN behind pfSense that the phone should reach |
| Phone (Tailscale IP) | `100.87.39.25` | the address Tailscale gave the phone. Yours will be different |

## What you need
- pfSense CE on a machine with:
  - a 64-bit amd64 (x86-64) CPU
  - 1 GB of RAM or more
  - an 8 GB or larger disk
  - one or more supported network cards
- A free Tailscale account (https://tailscale.com)
- The Tailscale app on your phone

The video uses pfSense CE and Tailscale 1.26.2.

## 1. Create an auth key
1. Go to https://tailscale.com and log in (the video signs in with Google).
2. **Settings → Keys → Generate auth key...**
3. Turn on **Reusable**. Leave **Ephemeral** and **Tags** off.
4. Click **Generate key**, copy the key and paste it into Notepad for now.

Treat this key like a password. Anyone who has it can add a machine to your Tailscale network.

> Note (2026): an auth key expires after at most 90 days (1 to 90, you pick when you create it).

## 2. Install the Tailscale package
In pfSense: **System → Package Manager → Available Packages** → search `Tailscale` → **Install** → **Confirm**.

The package is `pfSense-pkg-Tailscale`. When it's done, it shows up under **VPN → Tailscale**.

## 3. Authentication and settings
**VPN → Tailscale → Authentication**

| Field | Value |
|---|---|
| Login Server | `https://controlplane.tailscale.com` (leave the default) |
| Pre-authentication Key | `YOUR-AUTH-KEY` (the key from step 1) |

Click **Save**.

**VPN → Tailscale → Settings**

| Field | Value |
|---|---|
| Enable | ✔ Enable Tailscale |
| Listen Port | `41641` (leave the default) |
| Keep Configuration | ✔ Enable (leave the default) |
| Advertise Exit Node | ✔ |
| Accept Subnet Routes | ✔ |

- **Advertise Exit Node** lets other devices send all their internet traffic through pfSense if you want that later.
- **Accept Subnet Routes** lets pfSense reach networks that another Tailscale router (for example at another location) shares.

Both depend on your setup. The video ticks both.

## 4. Advertise your network (subnet router)
Still on the **Settings** tab. This is the network behind pfSense that your Tailscale devices will be able to reach:

| Field | Value |
|---|---|
| Advertised Routes | `10.37.0.0/20` (change to yours) |
| Description | `Server Vlan` |

To share more networks, click **Add** and add a line for each one.

Logging (optional, the video turns it on to see detailed logs):

| Field | Value |
|---|---|
| Syslog Logging | ✔ Enable syslog output |
| Syslog Settings | `DEBUG`, `LOG_DAEMON` |

Click **Save**. The top of the page should now say **Tailscale is online.**

## 5. Connect your phone
1. Install the Tailscale app (App Store or Google Play).
2. Open it → **Get started** → sign in with the same account as in step 1.

The phone joins your Tailscale network by itself.

## 6. Approve the routes in the admin console
Advertised routes do nothing until you approve them.

1. Open the Tailscale admin console → **Machines**. You should see `pfsense` (with **Subnets** and **Exit Node** labels) and your phone.
2. On the `pfsense` line, click **...** → **Edit route settings**.
3. Turn on the subnet route `10.37.0.0/20`.
4. **Use as exit node**: the video turns it off, so the phone keeps using its own internet connection and only home traffic goes through pfSense. Turn it on only if you want all the phone's traffic to go through your home.

You should see **Updated route settings**.

> Note (2026): in the current admin console you approve routes by selecting the machine on the **Machines** page and opening its **Subnets** section, then selecting the routes and **Save**.

## 7. Ping test both ways
**From the phone** (any ping app), ping pfSense's LAN address:
```
10.37.0.1
```
You should get replies.

**From pfSense**: **Diagnostics → Ping** → Hostname = the phone's Tailscale IP (shown in the admin console under **Machines**), for example:
```
100.87.39.25
```
Click **Ping**. You should see `3 packets transmitted, 3 packets received, 0.0% packet loss`.

## Links
- Tailscale: https://tailscale.com
- Tailscale subnet routers: https://tailscale.com/kb/1019/subnets
- pfSense: https://www.pfsense.org

More pfSense:
- pfSense gaming / traffic shaping: https://youtu.be/828R_GlPiao
- pfSense to Azure site-to-site VPN: https://youtu.be/sEgKqQ8Gm3Y
- pfSense to AWS site-to-site VPN: https://youtu.be/p83RmeT2Q-A
