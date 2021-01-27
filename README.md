# OpenVPN and LDAP authentication

This step-by-step guide will help configure LDAP authentication in OpenVPN server, create and configure self service password site.  
I already wrote this documentation for work but I'd like to share it so that other people can save their time on research for this.  

## Requirements  
* LDAP server
* Ubuntu machine(>= 16.04) for OpenVPN
* Ubuntu machine(>= 16.04) for [PWM](https://github.com/pwm-project/pwm)

## LDAP authentication in OpenVPN server  
1. Install packages below  
    - openvpn  
    - openvpn-auth-ldap  
2. Make sure ports for SSH, HTTPS and OpenVPN service in server are allowed  
```
sudo ufw allow ssh  
sudo ufw allow https  
sudo ufw allow 1194/udp  
sudo ufw enable  
```
3. Once all packages are installed in the VPN server, copy conf/auth-ldap.conf file to vpnserver:/etc/openvpn/auth/auth-ldap.conf. LDAP server's FQDN, bindDN user/password, and domain component should be changed  
4. Add a line below in /etc/openvpn/server.conf
```
plugin /usr/lib/openvpn/openvpn-auth-ldap.so /etc/openvpn/auth/auth-ldap.conf
```
5. Restart openvpn service and check the LDAP connection between the VPN server and LDAP server
```
sudo systemctl restart openvpn
sudo netstat -puant | grep 389
```
