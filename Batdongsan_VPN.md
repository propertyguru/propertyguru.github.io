<!-- this file can be viewed online at https://propertyguru.github.io/Batdongsan_VPN/ -->
# VPN Client

Delete the zip file and the email after confirming the VPN connection works. The `.ovpn` files contain a secret which should not leave the device where it's being used. Also don't backup the `.ovpn` files. In case the file is lost because of reinstall, hardware failure or replacement, ask for a new VPN certificate. Do not copy the certificate to another device, ask for a separate certificate for each device. In case the device is stolen or lost, or the VPN is not being used any more, inform <admin@batdongsan.com.vn> to revoke the certificate.

## Windows
1. Install OpenVPN from <https://openvpn.net/community-downloads/>.
Choose `Windows installer (NSIS)`.  
During the installation accept the default options, we don't need more than that, and less than that will probably not work.  
2. Start the OpenVPN GUI, the icon was added to your desktop ![OpenVPN icon](icon-3.png)
3. If this is your first time using OpenVPN, then the GUI will complain that "No readable connection profiles (config files) found. This is normal.
4. Now you need to tell OpenVPN to use the `propertyguru-vpn-hanoi-datacenter.ovpn` config file you received. There are 2 ways to do this:
   * Copy the `.ovpn` file from the zip file to the `%userprofile%\OpenVPN\config` folder.
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
4. Choose the `.ovpn` file which you extracted from the zip file in the email.
5. On the `IPv4 Settings` tab choose `Routes...` and make sure the `Use this connection only for resources on its network` is checked. By default it's unchecked and then NetworkManager adds a default route over the VPN which means all your internet traffic will go over the VPN. This is probably not what you want. When it's checked, then only the traffic which actually needs VPN will be routed over to the office network.  
  ![NetworkManager IPv4 Routes](NetworkManager%20IPv4%20Routes.png)
6. On the `General` tab uncheck  
- [ ] All users may connect to this network
8. `Save` the new VPN connection  
![NetworkManager Save](NetworkManager%20Save%20Button.png)

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
