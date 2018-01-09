# openvpn-manage

`openvpn-server-setup` designed to help OpenVPN server setup on `CentOS 6`, `CentOS 7` or `Ubuntu Xenial 16.04`.

It can be used for stock openvpn server configuration.

Finally it setup dual TCP/UDP openvpn server running in chroot environment.

You can put `openvpn-server-setup` to remote host and run it:
```bash
wget https://github.com/vmspike/openvpn-manage/raw/master/openvpn-server-setup
sudo bash ./openvpn-server-setu
```

Options `mssfix` (default) or `link-mtu` can be specified to automatically set confguration type during setup:
  `mssfix` generally more stable but less resistant to openvpn connection tracking by web sites, `link-mtu` usage also require manually set fixed MTU on Windows client hosts for TAP adapter because automatic change does not supported by tap adapter on Windows hosts.
You can change this behavior after creation by modifying server and client configs (comment/uncomment corresponding lines).

For other customizations you still need manually tune configuration after setup.

`openvpn-manage` tool embedded to `openvpn-server-setup` script as well as other configuration files and scripts.

External resources required for the script (except public repos and keys):
- https://github.com/vmspike/openvpn-manage/raw/master/easyrsa3.zip
- https://busybox.net/downloads/binaries/1.27.1-i686/busybox_ASH
- https://busybox.net/downloads/binaries/1.27.1-i686/busybox_IPROUTE

Other repo files are deprecated.
