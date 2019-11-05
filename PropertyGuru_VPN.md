<!-- this file can be viewed online at https://propertyguru.github.io/PropertyGuru_VPN/ -->
# VPN Client

When you want to connect to the PropertyGuru Singapore or Thailand office over VPN from home or somewhere else.

Please add both the Singapore and Thailand config files, even if you intend to connect to only one, as in case the VPN in one offices is down then it will be possible to use the other office as a fallback.

Delete the zip file and the email after confirming the VPN connection works. The `.ovpn` files contain a secret which should not leave the device where it's being used. Also don't backup the `.ovpn` files. In case the file is lost because of reinstall, hardware failure or replacement, ask for a new VPN certificate. Do not copy the certificate to another device, ask for a separate certificate for each device. In case the device is stolen or lost, or the VPN is not being used any more, inform systems@propertyguru.com.sg to revoke the certificate.

The VPN email will also contain a mysql zip file, this is to set up a connection to our mysql servers based on https://github.com/propertyguru/guruconf/blob/master/documentation/MySQL/mysql-workbench.md. If you did not request mysql access, ignore it. This is a work in progress and the mysql zip is sent for every VPN request, even if you didn't request mysql access.

## Windows
1. Install OpenVPN from <https://openvpn.net/community-downloads/>.
Choose `Windows installer (NSIS)`.  
During the installation accept the default options, we don't need more than that, and less than that will probably not work.  
2. Start the OpenVPN GUI, the icon was added to your desktop ![OpenVPN icon](icon-3.png)
3. If this is your first time using OpenVPN, then the GUI will complain that "No readable connection profiles (config files) found. This is normal.
4. Now you need to tell OpenVPN to use the `propertyguru-vpn-singapore.ovpn` and `propertyguru-vpn-thailand.ovpn` config files you received. There are 2 ways to do this:
   * Copy the `.ovpn` files from the zip file to the `%userprofile%\OpenVPN\config` folder.
   * Right click on the OpenVPN tray icon, choose "Import file..." and select the `.ovpn` file from the zip
file:  
![OpenVPN Import file...](OpenVPN%20Import%20file....png)
5. Now when you right click on the OpenVPN tray icon, you will see a "Connect" item too:  
![OpenVPN Connect](OpenVPN%20Connect.png)
6. If you have any issues connecting, then choose `View Log` from the menu and send the whole log to your friendly IT support.

## Ubuntu with NetworkManager
We are detailing the steps for Ubuntu, as that's the most common OS, and with NetworkManager because that's the most convenient way of doing it. It's also possible to set up the connection without NetworkManager, but then you will have to stop/start the vpn from the command line and you will have no GUI indication if the VPN is running or not.

This needs at least Ubuntu 18.04 LTS Bionic. The NetworkManager version in Ubuntu 16.04 LTS Xenial is too old and doesn't support the features we need.

1. Install the `network-manager-openvpn-gnome` package with your favourite package manager.
2. Click the NetworkManager tray icon and choose `Edit Connections...`  
  ![NetworkManager Edit Connections...](NetworkManager%20Edit%20Connections....png) |
3. Choose `Import a saved VPN configuration...` as the connection type  
![NetworkManager Connection Type](NetworkManager%20Connection%20Type.png) |
4. Choose the `propertyguru-vpn-singapore.ovpn` file which you extracted from the zip file in the email.
5. On the `IPv4 Settings` tab choose `Routes...` and make sure the `Use this connection only for resources on its network` is checked. By default it's unchecked and then NetworkManager adds a default route over the VPN which means all your internet traffic will go over the VPN. This is probably not what you want. When it's checked, then only the traffic which actually needs VPN will be routed over to the office network.  
  ![NetworkManager IPv4 Routes](NetworkManager%20IPv4%20Routes.png)
6. On the `General` tab uncheck  
- [ ] All users may connect to this network
7. Optional: NetworkManager 1.10.6 from Ubuntu Bionic does not support the `compress`, and ignores it. This is a problem because then the `push "compress lzo"` command from the server will also be ignored and the connection will fail. To work around this go to the `VPN` tab, click `Advanced...` and check:  
- [x] Use LZO data compression `adaptive`  
The downside is that if we change the compression on the server, that will not match the hardcoded value on the client any more, and the connection will fail. <!-- Check if it's fixed in the next Ubuntu release. -->
8. `Save` the new VPN connection  
![NetworkManager Save](NetworkManager%20Save%20Button.png)
9. Repeat the above steps for the `propertyguru-vpn-thailand.ovpn` file.

## MacOS
The recommended way is to use Tunnelblick, as that gives a GUI status indicator to show when the VPN is connected. It's also possible to use the OpenVPN command line interface, but that has no status indicator, so it's less convenient.

### Mac OS with Tunnelblick GUI
Follow openvpn protocol guide to setup VPN on macos as follows: <http://www.vpngate.net/en/howto_openvpn.aspx#mac>

Tunnelblick will show a warning:
<blockquote>
  This computer's apparent public IP address was not different after connecting to propertyguru-vpn-singapore. It is still W.X.Y.Z.
  
  This may mean that your VPN is not configured correctly.
</blockquote>
This is because we don't route everything over the VPN, only the traffic which goes to our internal network, and some public IPs, which have source IP rules. So just select `Do not warn about this again for any configuration` and press `OK`.

## Android
1. Install the [OpenVPN Connect app from the Play Store](https://play.google.com/store/apps/details?id=net.openvpn.openvpn)
2. TODO Please help updating this guide by editing https://github.com/propertyguru/propertyguru.github.io/blob/master/PropertyGuru_VPN.md
3. Profit!

## iOS
TODO Please help updating this guide by editing https://github.com/propertyguru/propertyguru.github.io/blob/master/PropertyGuru_VPN.md

Because apple store doesn't allow GPL the OpenVPN app doesn't have compression support. That means the VPN connection to the server will be successful, but no traffic can get through and the server log has a lot of `Bad LZO decompression header byte: 251`. This was worked around in the Thailand office by switching off compression for clients which don't support it. But the OpenVPN server version in Singapore is too old to be able to do it. The result is that currently it's not possible to connect to the Singapore VPN with iOS. But it's possible to connect to the Thailand VPN.

## FAQ
### How to determine the VPN access was successful or not?
A good way is to check the routing table. We push more than 100 routes over the VPN, so if you see a lot of routes, then the VPN connection was successful.

Another way to verify is to try to open the staging site, because that is accessible only from the offices or over the VPN, for example https://www.staging.propertyguru.com.sg/. If you see the website and you are not in the office, then the VPN connection was successful. If you see `403 Forbidden`, then you are not accessing the page over the VPN.

### What traffic is sent over the VPN?
The list of static routes which we push from the VPN server can be found by looking for `push "route` in https://github.com/propertyguru/puppet/tree/master/modules/osg_uservpn/files/etc_openvpn. Additionally there are DNS names which get resolved every hour and pushed over the VPN connection too, those are in https://github.com/propertyguru/puppet/blob/master/modules/osg_uservpn/files/x-openvpn-scripts-client-connect-generate-dns-cache.

By default we only send traffic over the VPN which needs VPN. We don't set the default route to go over the VPN, so only traffic to the propertyguru websites should go over the VPN, the traffic to other websites will avoid the VPN. We don't want to slow down traffic to the rest of the internet by forcing it to go over the VPN. We also don't want unnecessary traffic to go over the office internet connections. But we don't prevent the VPN to be used as a default gateway, in case it's needed.

For Linux NetworkManager to use the VPN for all traffic: open the VPN connection settings, and under `IPv4 Settings -> Routes...` switch off `Use this connection only for resources on its network`. This will add a default route over the VPN.
