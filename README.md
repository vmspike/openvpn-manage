# openvpn-manage

## Description
The script `openvpn-server-setup` designed to setup dual TCP/UDP OpenVPN server running in chroot environment on single run and then have ability to manage unlimited number of vpn users for free by `openvpn-manage`.

Supported OS: `CentOS 6`, `CentOS 7`, `Ubuntu Xenial 16.04`.

It can be used for quick stock openvpn server installation and initial (or final) configuration.
For customizations you can manually tune configuration files after setup.

Default setup creates server running on ports 443/tcp and 443/udp.

## Usage
```bash
# Download the script
wget https://github.com/vmspike/openvpn-manage/raw/master/openvpn-server-setup

# Review the content if you're paranoid and launch:
sudo bash ./openvpn-server-setup
# OR if you want to protect yourself from VPN tracking at the cost of performance, bandwidth, latency and stability
sudo bash ./openvpn-server-setup --mssfix 0

# If the script is unsure about some actions it will ask you interactively.
# Review the script output to be sure that it completed successfully


# After all apply customization if you need and create user(s)
openvpn-manage create John_Smith
```

`openvpn-server-setup` can be called with additional options: `[-i IFNAME] [--mssfix [VAL]] [--fragment [VAL]]`

By default this script creates configurations with default `mssfix` and `fragment` options
which is fine in most cases. BTW you can set the value you want by
commandline option, particularly `mssfix 0` which can lead to high packets
fragmentation which by-turn can lead to bandwidth and latency issues, but
it's much more resistant to VPN-tracking by external web-sites and do not
requre manual TAP adapter MTU fixing on Windows (which is required by `tun-mtu`
or `link-mtu`).

In all cases you can change this behavior after creation by modifying server and client configs.

Network interface defaults to `eth0` if not specified by `-i`.

After setup `openvpn-manage` tool will be available. Run it without arguments for help. Its content embedded to `openvpn-server-setup:OPENVPN_MANAGE` variable on top of the script.

## Customization
To customize setup you can review/change these files:
- /etc/openvpn/udp.server.conf
- /etc/openvpn/tcp.server.conf
- /etc/openvpn/chroot/users-config
- /etc/openvpn/server/scripts/openvpn-manage.conf
- /etc/openvpn/easy-rsa/easyrsa3/vars
- /etc/openvpn/easy-rsa/easyrsa3/openssl-1.0.cnf
- /etc/rc.local

When some users already created:
- /etc/openvpn/chroot/ccd/*

You also could want to change firewall (iptables) setup.
To see existing rules run:
```
for TBL in raw mangle nat filter; do echo ==${TBL}==; sudo iptables -vnL -t ${TBL}; echo; done
```

## Clients setup
`openvpn-manage` creates single configuration file with all certs embedded (`.ovpn` or `.conf`), after that you just have to download it and import to your device.

**Note:** you can't use the same openvpn client config on different devices simultaneously (yet, if you have not adopt configuration to allow it), it will lead to reconnecting loop on all your devices. If you have multiple devices you want to connect to VPN simultaneously you have to create additional vpn config(s)

OpenVPN version have to be >=2.4, it will not work with 2.3 release!

General instructions for various OSes:
- Windows

    - Download from [here](https://openvpn.net/index.php/open-source/downloads.html) and install latest openvpn;
    - If your openvpn config has `.conf` extension (Linux/MacOS specific), change it to `.ovpn` (Windows specific);
    - Put config to `C:\\Users\YOURUSERNAME\OpenVPN\config\` for specific user or to `C:\\Program Files\OpenVPN\config\` for all users;
    - Be sure your localhost time is in sync with NTP server (see time settings);
    - Start OpenVPN GUI and connect, you'll see green icon in the tray on success and new tun/tap interface will be available;
    - Optionally ping VPN server to be sure it works properly: press `winkey+r` -> type `cmd` -> type `ping 172.31.240.1`

    If you use configuration with `link-mtu` you additionally have to:
    - Check tun-mtu value in openvpn client log on first connection attempt. The line looks like this: `Tue Jun 29 11:38:00 2018 WARNING: normally if you use --mssfix and/or --fragment, you should also set --tun-mtu 1500 (currently it is 1296)`
    - Set the same value for MTU in TAP adapter settings (1296 in this example): `Control Panel` -> `Network and Internet` -> `Network Connnections` -> `Tap-Windows Adapter V9` -> `Properties` -> `Setup` -> `Additional settings` -> `MTU`
    - Close openvpn connection and connect again, to apply MTU changes.

- Linux

    - Add openvpn repository following the [instructions](https://community.openvpn.net/openvpn/wiki/OpenvpnSoftwareRepos) and install/upgrade openvpn;
    - Check version: `openvpn --version | head -1` (have to be >=2.4);
    - Check if your localhost time is in sync with NTP server (example for systemd OS): `timedatectl`;
    - Connect: `sudo openvpn /path/to/XXXXX-Username.conf`
    - Check that it works fine, review the log and try to ping VPN server: `ping -c4 172.31.240.1`
    - Optionally you can setup it as a service with autostart:
        - Close current connection (press `ctrl+c`);
        - Be sure the config extension is `.conf`;
        - Put the config to `/etc/openvpn/`;

        - Set `AUTOSTART="all"` in `/etc/default/openvpn` and apply changes
        (if any) for systemd: `sudo systemctl daemon-reload`;

        - Start service `sudo systemctl start openvpn@XXXXX-Username.service` (for systemd) or `sudo service openvpn start XXXXX-Username` (for SysV/Upstart, config name is optional if only one config exists).

- MacOS

    via Tunnelblick:
    - [Download](https://tunnelblick.net/downloads.html) and install;
    - Set few options in `VPN Details...` -> `Configurations` -> choose your config:
        - OpenVPN version: `2.4.* - OpenSSL v1.*`;

    You can also install console openvpn client via `brew` or `macports` and use it similar way as on Linux.

- Android

    - Change config extension from `conf` to `ovpn`
    - Import openvpn config to one of openvpn apps, for example:
        - [OpenVPN Connect](https://play.google.com/store/apps/details?id=net.openvpn.openvpn) - for simple cases, fast but proprietary;
        - [OpenVPN for Android](https://play.google.com/store/apps/details?id=de.blinkt.openvpn) - for advanced usage, slower but free/open_source and highly configurable.

- iOS

    - Use [OpenVPN Connect](https://itunes.apple.com/us/app/openvpn-connect/id590379981) or other app if available and setup similar way as on Android.

## Troubleshooting
- Cannot connect to the server after setup

Be sure that openvpn server ports are open on the host and router above it if any, for example on AWS you have to setup security group(s) on web UI to open ports on your new VM.


- My OS is unsupported

Feel free to adopt the script to your OS, PR's are welcome. Or ask for support for specific OS, I'll consider it's addition.


- My Android/iOS client failed to connect with `AUTH_FAILED` error

"OpenVPN Connect" could [send incorrect information](https://community.openvpn.net/openvpn/ticket/816) about enabled options, it leads to `AUTH_FAILED` if `--opt-verify` enabled on server side.
By default `opt-verify` enabled only on udp server config.
So use one of the ways to workaround it:

    - comment out line with `opt-verify` on server udp config and restart udp vpn server

    - comment out `udp` connection block in client config to force `tcp` (with `opt-verify` already disabled)

    - use another client instead, e.g. `OpenVPN for Android`


## Other notes
External resources required for the script to be available (except public repos and keys):
- https://github.com/vmspike/openvpn-manage/raw/master/easyrsa3.zip
- https://busybox.net/downloads/binaries/1.27.1-i686/busybox_ASH
- https://busybox.net/downloads/binaries/1.27.1-i686/busybox_IPROUTE

Other repo files are deprecated.
