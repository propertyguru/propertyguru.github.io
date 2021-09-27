<!-- this file can be viewed online at https://propertyguru.github.io/PropertyGuru_VPN/ -->
# VPN Client

When you want to connect to the PropertyGuru Singapore or Thailand office over VPN from home or somewhere else. For the system level documentation see <https://propertyguru.atlassian.net/wiki/display/IOD/VPN+Infrastructure>.

Please add both the Singapore and Thailand config files, even if you intend to connect to only one, as in case the VPN in one offices is down then it will be possible to use the other office as a fallback.

Delete the zip file and the email after confirming the VPN connection works. The `.ovpn` files contain a secret which should not leave the device where it's being used. Also don't backup the `.ovpn` files. In case the file is lost because of reinstall, hardware failure or replacement, ask for a new VPN certificate. Do not copy the certificate to another device, ask for a separate certificate for each device. In case the device is stolen or lost, or the VPN is not being used any more, inform <systems-sg@propertyguru.com.sg> to revoke the certificate.

The VPN email will also contain a mysql zip file, this is to set up a connection to our mysql servers based on <https://github.com/propertyguru/guruconf/blob/master/documentation/MySQL/mysql-workbench.md>. If you did not request mysql access, ignore it. This is a work in progress and the mysql zip is sent for every VPN request, even if you didn't request mysql access.

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

NetworkManager parses the openvpn configuration file and based on that creates a NetworkManager configuration file. The NetworkManager configuration is saved to `/etc/NetworkManager/system-connections/propertyguru-vpn-singapore.nmconnection` (and equivalent for Thailand). The certificates will be extracted from the ovpn file and placed in `$HOME/.cert/nm-openvpn/` and named `propertyguru-vpn-singapore-ca.pem` `propertyguru-vpn-singapore-cert.pem` and `propertyguru-vpn-singapore-key.pem` (and equivalent for Thailand).

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
2. TODO Please help updating this guide by editing <https://github.com/propertyguru/propertyguru.github.io/blob/master/PropertyGuru_VPN.md>
3. Profit!

## Apple iOS phones
1. Download “[OpenVPN Connect](https://apps.apple.com/th/app/openvpn-connect/id590379981)” from the Appstore.
2. Connect your iPhone with your Mac. Open “Finder”, select your phone from the left panel. Then click “Files” from the upper top right.
3. Drag unzipped certificate files and drop into “OpenVPN”
4. Open the “OpenVPN” app from your phone. You will see the certificate that you just added. 
5. Click the “Add” button.

Because apple store doesn't allow GPL the OpenVPN app doesn't have compression support. That means the VPN connection to the server will be successful, but no traffic can get through and the server log has a lot of `Bad LZO decompression header byte: 251`. This was worked around in the Thailand office by switching off compression for clients which don't support it. But the OpenVPN server version in Singapore is too old to be able to do it. The result is that currently it's not possible to connect to the Singapore VPN with iOS. But it's possible to connect to the Thailand VPN.

## FAQ

### Where is the password?
The initial ZIP password sent over slack is an ephemeral message, it's not stored on the slack servers, or the slack client and disappears after slack disconnects. If the password disappears before it's used to unpack the ZIP file, then request a new one. If the person who sent the VPN certificate still has the SSH window open, it's possible to resend the password. Otherwise the certificate will be revoked and a new certificate will be sent.

### How to determine the VPN access was successful or not?
A good way is to check the routing table. We push more than 100 routes over the VPN, so if you see a lot of routes, then the VPN connection was successful.

Another way to verify is to try to open the staging site, because that is accessible only from the offices or over the VPN, for example <https://www.staging.propertyguru.com.sg/>. If you see the website and you are not in the office, then the VPN connection was successful. If you see `403 Forbidden`, then you are not accessing the page over the VPN.

### What traffic is sent over the VPN?
The list of static routes which we push from the VPN server can be found by looking for `push "route` in <https://github.com/propertyguru/puppet/tree/master/modules/osg_uservpn/files/etc_openvpn>. Additionally there are DNS names which get resolved every hour and pushed over the VPN connection too, those are in <https://github.com/propertyguru/puppet/blob/master/modules/osg_uservpn/files/x-openvpn-scripts-client-connect-generate-dns-cache>.

By default we only send traffic over the VPN which needs VPN. We don't set the default route to go over the VPN, so only traffic to the propertyguru websites should go over the VPN, the traffic to other websites will avoid the VPN. We don't want to slow down traffic to the rest of the internet by forcing it to go over the VPN. We also don't want unnecessary traffic to go over the office internet connections. But we don't prevent the VPN to be used as a default gateway, in case it's needed.

For Linux NetworkManager to use the VPN for all traffic: open the VPN connection settings, and under `IPv4 Settings -> Routes...` switch off `Use this connection only for resources on its network`. This will add a default route over the VPN.

### VPN not working after 2021-09-30
If you got your VPN configuration file before 2021-09-21 and it stopped working exactly at 2021-09-30 22:01:15 SGT (21:01:15 ICT), then it's because of the [IdentTrust DST Root CA X3 certificate expiration](https://scotthelme.co.uk/lets-encrypt-old-root-expiration/).

To verify if you are using the old CA certificate you can look at the `propertyguru-vpn-singapore.ovpn` and `propertyguru-vpn-thailand.ovpn` files and if you see this, then you have the old CA:
```
<ca>
-----BEGIN CERTIFICATE-----
MIIDSjCCAjKgAwIBAgIQRK+wgNajJ7qJMDmGLvhAazANBgkqhkiG9w0BAQUFADA/
```

The easiest way to fix this is to request a new VPN certificate. If you don't want to wait for that and are comfortable with editing configuration files, then in the OpenVPN configuration file replace the text between `<ca>` and `</ca>` with the following:
```
<ca>
-----BEGIN CERTIFICATE-----
MIIFazCCA1OgAwIBAgIRAIIQz7DSQONZRGPgu2OCiwAwDQYJKoZIhvcNAQELBQAw
TzELMAkGA1UEBhMCVVMxKTAnBgNVBAoTIEludGVybmV0IFNlY3VyaXR5IFJlc2Vh
cmNoIEdyb3VwMRUwEwYDVQQDEwxJU1JHIFJvb3QgWDEwHhcNMTUwNjA0MTEwNDM4
WhcNMzUwNjA0MTEwNDM4WjBPMQswCQYDVQQGEwJVUzEpMCcGA1UEChMgSW50ZXJu
ZXQgU2VjdXJpdHkgUmVzZWFyY2ggR3JvdXAxFTATBgNVBAMTDElTUkcgUm9vdCBY
MTCCAiIwDQYJKoZIhvcNAQEBBQADggIPADCCAgoCggIBAK3oJHP0FDfzm54rVygc
h77ct984kIxuPOZXoHj3dcKi/vVqbvYATyjb3miGbESTtrFj/RQSa78f0uoxmyF+
0TM8ukj13Xnfs7j/EvEhmkvBioZxaUpmZmyPfjxwv60pIgbz5MDmgK7iS4+3mX6U
A5/TR5d8mUgjU+g4rk8Kb4Mu0UlXjIB0ttov0DiNewNwIRt18jA8+o+u3dpjq+sW
T8KOEUt+zwvo/7V3LvSye0rgTBIlDHCNAymg4VMk7BPZ7hm/ELNKjD+Jo2FR3qyH
B5T0Y3HsLuJvW5iB4YlcNHlsdu87kGJ55tukmi8mxdAQ4Q7e2RCOFvu396j3x+UC
B5iPNgiV5+I3lg02dZ77DnKxHZu8A/lJBdiB3QW0KtZB6awBdpUKD9jf1b0SHzUv
KBds0pjBqAlkd25HN7rOrFleaJ1/ctaJxQZBKT5ZPt0m9STJEadao0xAH0ahmbWn
OlFuhjuefXKnEgV4We0+UXgVCwOPjdAvBbI+e0ocS3MFEvzG6uBQE3xDk3SzynTn
jh8BCNAw1FtxNrQHusEwMFxIt4I7mKZ9YIqioymCzLq9gwQbooMDQaHWBfEbwrbw
qHyGO0aoSCqI3Haadr8faqU9GY/rOPNk3sgrDQoo//fb4hVC1CLQJ13hef4Y53CI
rU7m2Ys6xt0nUW7/vGT1M0NPAgMBAAGjQjBAMA4GA1UdDwEB/wQEAwIBBjAPBgNV
HRMBAf8EBTADAQH/MB0GA1UdDgQWBBR5tFnme7bl5AFzgAiIyBpY9umbbjANBgkq
hkiG9w0BAQsFAAOCAgEAVR9YqbyyqFDQDLHYGmkgJykIrGF1XIpu+ILlaS/V9lZL
ubhzEFnTIZd+50xx+7LSYK05qAvqFyFWhfFQDlnrzuBZ6brJFe+GnY+EgPbk6ZGQ
3BebYhtF8GaV0nxvwuo77x/Py9auJ/GpsMiu/X1+mvoiBOv/2X/qkSsisRcOj/KK
NFtY2PwByVS5uCbMiogziUwthDyC3+6WVwW6LLv3xLfHTjuCvjHIInNzktHCgKQ5
ORAzI4JMPJ+GslWYHb4phowim57iaztXOoJwTdwJx4nLCgdNbOhdjsnvzqvHu7Ur
TkXWStAmzOVyyghqpZXjFaH3pO3JLF+l+/+sKAIuvtd7u+Nxe5AW0wdeRlN8NwdC
jNPElpzVmbUq4JUagEiuTDkHzsxHpFKVK7q4+63SM1N95R1NbdWhscdCb+ZAJzVc
oyi3B43njTOQ5yOf+1CceWxG1bQVs5ZufpsMljq4Ui0/1lvh+wjChP4kqKOJ2qxq
4RgqsahDYVvTH9w7jXbyLeiNdd8XM2w9U/t7y0Ff/9yi0GE44Za4rF2LN9d11TPA
mRGunUHBcnWEvgJBQl9nJEiU0Zsnvgc/ubhPgXRR4Xq37Z0j4r7g1SgEEzwxA57d
emyPxgcYxn/eR44/KJ4EBs+lVDR3veyJm+kXQ99b21/+jh5Xos1AnX5iItreGCc=
-----END CERTIFICATE-----
</ca>
```

#### Explanation
Our VPN server uses Let's Encrypt to get it's own TLS certificate. This is used by the VPN client to verify it's really talking to the PropertyGuru VPN servers, to avoid MiTM attacks. But since OpenVPN does not use the operating system's trusted CA certificates to verify the VPN server, we have to put the CA certificate which signed the Let's Encrypt certificate into the VPN configuration file. This certificate is different from the VPN client certificate, which is also in the configuration file, but in the `<cert>` tag. That has it's own expiration and is signed by a different CA.

This issue was fixed on the VPN server in [commit fa5ecdd](https://github.com/propertyguru/puppet/commit/fa5ecdddb96726ff1590289006890c5ca6a9d4f7). VPN configuration files sent on 2021-09-21 (and later) already embed the new CA certificate and are not affected.

## Improvements
Please help updating this guide by editing <https://github.com/propertyguru/propertyguru.github.io/blob/master/PropertyGuru_VPN.md>
