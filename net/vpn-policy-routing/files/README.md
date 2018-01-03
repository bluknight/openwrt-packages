# VPN Policy-Based Routing

## Description
This service allows you to define rules (policies) for routing traffic via WAN or your Openconnect, OpenVPN or Wireguard tunnels. Policies can be set based on any combination of local/remote ports, local/remote IPv4 or IPv6 addresses/subnets or domains. This service supersedes the [VPN Bypass](https://github.com/openwrt/packages/blob/master/net/vpnbypass/files/README.md) service, by supporting IPv6 and by allowing you to set explicit rules not just for WAN gateway (bypassing OpenVPN tunnel), but for Openconnect, OpenVPN and Wireguard tunnels as well.

## Features

#### Gateways/Tunnels
- Any policy can target either WAN, an OpenVPN tunnel or a Wireguard interface.
- Multiple Openconnect interfaces supported (with protocol names openconnect\*).
- Multiple OpenVPN tunnels supported (with device names tun\* or tap\*).
- Multiple Wireguard interfaces supported (with protocol names wireguard\*).

#### IPv4/IPv6/Port-Based Policies
- Policies based on local names, IPs or subnets. You can specify a single IP (as in ```192.168.1.70```) or a local subnet (as in ```192.168.1.81/29```) or a local device name (as in ```nexusplayer```). IPv6 addresses are also supported.
- Policies based on local ports numbers. Can be set as an individual port number (```32400```), comma-separated list (```80,8080```) or a range (```5060-5061```). Limited to 15 ports per policy.
- Policies based on remote IPs/subnets or domain names. Same format/syntax as local IPs/subnets.
- Policies based on remote ports numbers. Same format/syntax and restrictions as local ports.
- You can mix and match IP addresses/subnets and device (or domain) names in one field separating them by space or semi-colon (like this: ```66.220.2.74 he.net tunnelbroker.net```).

#### DSCP-tag Based Policies
You can also set policies for traffic with specific DSCP tag. Instructions for tagging specific app traffic in Windows 10 can be found [here](http://serverfault.com/questions/769843/cannot-set-dscp-on-windows-10-pro-via-group-policy).

#### Strict enforcement
- Supports strict policy enforcement, even if the policy gateway is down -- resulting in network being unreachable for specific policy (enabled by default).

#### Use DNSMASQ
- Service can be set to utilize ```dnsmasq```'s ```ipset``` support. This requires the ```dnsmasq-full``` to be installed (see [How to install dnsmasq-full](#how-to-install-dnsmasq-full)) and it significantly improves the start up time because dnsmasq resolves the domain names and adds them to ipsets in the background. Another benefit if using ```dnsmasq```'s ```ipset``` is that it also automatically adds third-level domains to the ipset; if ```domain.com``` is added to the policy, this policy will affect all ```*.domain.com``` subdomains.

#### Customization
- Can be fully configured with uci commands or by editing ```/etc/config/vpn-policy-routing``` file.
- Has a companion package (luci-app-vpn-policy-routing) so everything can be configured with Web UI.

#### Other Features
- Doesn't stay in memory, creates the ip tables and iptables rules which are automatically updated when supported/monitored interface changes.
- Proudly made in Canada, using locally-sourced electrons.

## Screenshot (luci-app-vpn-policy-routing)
![screenshot](https://raw.githubusercontent.com/stangri/screenshots/master/vpn-policy-routing/screenshot01.png "screenshot")

## Requirements
This service requires the following packages to be installed on your router: ```ipset```, ```resolveip```, ```kmod-ipt-ipset``` and ```iptables```.

To satisfy the requirements, connect to your router via ssh and run the following commands:
```sh
opkg update; opkg install ipset resolveip kmod-ipt-ipset iptables
```

#### How to install dnsmasq-full
If you want to use ```dnsmasq```'s ```ipset``` support, you will need to install ```dnsmasq-full``` instead of the ```dnsmasq```. To do that, connect to your router via ssh and run the following command:
```sh
opkg update; opkg remove dnsmasq; opkg install dnsmasq-full
```


#### Unmet dependencies
If you are running a development (trunk/snapshot) build of OpenWrt/LEDE Project on your router and your build is outdated (meaning that packages of the same revision/commit hash are no longer available and when you try to satisfy the [requirements](#requirements) you get errors), please flash either current LEDE release image or current development/snapshot image.

## How to install
Please make sure that the [requirements](#requirements) are satisfied and install ```vpn-policy-routing``` and ```luci-app-vpn-policy-routing``` from Web UI or connect to your router via ssh and run the following commands:
```sh
opkg update
opkg install vpn-policy-routing luci-app-vpn-policy-routing
```
If these packages are not found in the official feed/repo for your version of OpenWrt/LEDE Project, you will need to [add a custom repo to your router](#add-custom-repo-to-your-router) first.

#### Add custom repo to your router
If your router is not set up with the access to repository containing these packages you will need to add custom repository to your router by connecting to your router via ssh and running the following commands:

###### OpenWrt 15.05.1
```sh
opkg update; opkg install ca-certificates wget libopenssl
echo -e -n 'untrusted comment: public key 7ffc7517c4cc0c56\nRWR//HUXxMwMVnx7fESOKO7x8XoW4/dRidJPjt91hAAU2L59mYvHy0Fa\n' > /tmp/stangri-repo.pub && opkg-key add /tmp/stangri-repo.pub
! grep -q 'stangri_repo' /etc/opkg/customfeeds.conf && echo 'src/gz stangri_repo https://raw.githubusercontent.com/stangri/openwrt-repo/master' >> /etc/opkg/customfeeds.conf
opkg update
```

###### LEDE Project and OpenWrt 18.xx or later
```sh
opkg update; opkg install uclient-fetch libustream-mbedtls
echo -e -n 'untrusted comment: public key 7ffc7517c4cc0c56\nRWR//HUXxMwMVnx7fESOKO7x8XoW4/dRidJPjt91hAAU2L59mYvHy0Fa\n' > /tmp/stangri-repo.pub && opkg-key add /tmp/stangri-repo.pub
! grep -q 'stangri_repo' /etc/opkg/customfeeds.conf && echo 'src/gz stangri_repo https://raw.githubusercontent.com/stangri/openwrt-repo/master' >> /etc/opkg/customfeeds.conf
opkg update
```

## Default Settings
Default configuration has service disabled (use Web UI to enable/start service or run ```uci set vpn-policy-routing.config.enabled=1```) and has some example policies which can be safely deleted.

#### Additional settings
Some of the ```vpn-policy-routing``` settings are intentionally not exposed thru Web UI and require use of either ```uci``` commands or editing of ```/etc/config/vpn-policy-routing``` file on the router. The full list of configuration parameters of ```vpn-policy-routing.config``` section is below.

|Parameter|Type|Default|Comment|
| --- | --- | --- | --- |
|enabled|boolean|0|Enable/disable the ```vpn-policy-routing``` service.|
|strict_enforcement|boolean|1|Enforce policies when their gateway is down.|
|ipv6_enabled|boolean|1|Enable/disable IPv6 support.|
|ipset_enabled|boolean|1|Enable use of ipsets. Make sure the [requirements](#requirements) are met.|
|dnsmasq_enabled|boolean|1|Enable use of dnsmasq for ipsets. Assumes ```ipset_enabled=1``` Make sure the [requirements](#requirements) are met.|
|udp_proto_enabled|boolean|0|Add UDP protocol iptables rules for protocol policies with unset local addresses and either local or remote port set. Unless this variable is set to 1, only TCP protocol iptables rules are added. |
|forward_chain_enabled|boolean|0|Create and use a ```FORWARD``` chain in the mangle table. By default the ```vpn-policy-routing``` only creates and uses the ```PREROUTING``` chain. Use with caution.|
|input_chain_enabled|boolean|0|Create and use an ```INPUT``` chain in the mangle table. By default the ```vpn-policy-routing``` only creates and uses the ```PREROUTING``` chain. Use with caution.|
|output_chain_enabled|boolean|0|Create and use an ```OUTPUT``` chain in the mangle table. By default the ```vpn-policy-routing``` only creates and uses the ```PREROUTING``` chain. Policies in the OUTPUT chain will affect traffic from the router itself. All policies with unset local address will be duplicated in the ```OUTPUT``` chain. Use with caution.|
|verbosity|integer|2|Can be set to 0, 1 or 2 to control the console and system log output verbosity of the ```vpn-policy-routing``` service.|
|wan_tid|integer|201|Starting (WAN) Table ID number for tables created by the ```vpn-policy-routing``` service.|
|wan_mark|hexadecimal|0x010000|Starting (WAN) fw mark for marks used by the ```vpn-policy-routing``` service. High starting mark is used to avoid conflict with SQM/QoS, this can be changed by user. Change with caution together with ```fw_mask```.|
|fw_mask|hexadecimal|0xff0000|FW Mask used by the ```vpn-policy-routing``` service. High mask is used to avoid conflict with SQM/QoS, this can be changed by user. Change with caution together with ```wan_mark```.|
|icmp_interface|string||Set the default ICMP protocol interface. Requires ```output_chain_enabled=1```. Use with caution.|
|ignored_interface|string||Allows to specify the list of interface names to be ignored by the ```vpn-policy-routing``` service. Can be useful if running both VPN server and VPN client on the router.|


## Discussion
Please head to [LEDE Project Forum](https://forum.lede-project.org/t/openvpn-wireguard-policy-based-routing-web-ui/1422) for discussions of this service.

#### Getting help
If things are not working as intended, please include the output of ```/etc/init.d/vpn-policy-routing support``` with your post, as well as the output of ```/etc/init.d/vpn-policy-routing reload``` with verbosity setting set to maximum.
If you don't want to post the ```/etc/init.d/vpn-policy-routing support``` output in a public forum, there's a way to have the support details automatically uploaded to my account at paste.ee by running ```/etc/init.d/vpn-policy-routing support -p```. You need to have the following packages installed to enable paste.ee upload functionality: ```curl libopenssl ca-bundle```.
WARNING: while paste.ee uploads are unlisted, they are still publicly available.


## What's New
0.0.1:
- Initial release.

## Notes/Known Issues
- While you can select down/inactive OpenVPN/Wireguard tunnel in Web UI, the appropriate tunnel must be up/active for the policies to properly work without errors on service start.

- Service does not alter the default routing. Depending on your OpenVPN/Wireguard settings (and settings of the OpenVPN/Wireguard server you are connecting to), the default routing might be set to go via WAN or via OpenVPN/Wireguard tunnel. This service affects only routing of the traffic matching the policies. If you want to override default routing, consider adding the following to your OpenVPN tunnel(s) config:
```
option route_nopull '1'
```
<!-- option route '0.0.0.0 0.0.0.0'  -->
  and set the following for your Wireguard tunnel(s) config:
```
option route_allowed_ips '0'
```

## Thanks
I'd like to thank everyone who helped create, test and troubleshoot this service. Without contributions from [@hnyman](https://github.com/hnyman), [@dibdot](https://github.com/dibdot), [@danrl](https://github.com/danrl), [@tohojo](https://github.com/tohojo), [@cybrnook](https://github.com/cybrnook), [@nidstigator](https://github.com/nidstigator), [@AndreBL](https://github.com/AndreBL) and [@dz0ny](https://github.com/dz0ny) and rigorous testing by [@dziny](https://github.com/dziny), [@bluenote73](https://github.com/bluenote73) and   [@buckaroo](https://github.com/pgera) it wouldn't have been possible.
