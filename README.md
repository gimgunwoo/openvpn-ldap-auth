# OpenVPN and LDAP authentication

This step-by-step guide will help configure LDAP authentication in OpenVPN server, create and configure self service password site.  
I already wrote this documentation for work but I'd like to share it so that other people can save their time on research for this.  

## Table of contents
[Requirements](#Requirements)  
[LDAP authentication in OpenVPN server](#LDAP authentication in OpenVPN server)  

<a name="Requirements"/>
## Requirements  
* LDAP server
* Ubuntu machine(>= 16.04) for OpenVPN
* Ubuntu machine(>= 16.04) for [PWM](https://github.com/pwm-project/pwm)

<a name="LDAP authentication in OpenVPN server"/>
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
3. Once all packages are installed in the VPN server, create /etc/openvpn/auth/auth-ldap.conf file below. FQDN, bindDN user/password, and domain component should be changed accordingly  
<details>
  <summary>Expand</summary>
  ```
  <LDAP>
          # LDAP server URL
          URL             ldap://FQDN
  
          # Bind DN (If your LDAP server doesn't support anonymous binds)
          BindDN          uid=<binddn_user>,cn=users,cn=accounts,dc=<your_domain_components>
  
          # Bind Password
          Password        <password>
  
          # Network timeout (in seconds)
          Timeout         15
  
          # Enable Start TLS
          TLSEnable       no
  
          # Follow LDAP Referrals (anonymously)
          FollowReferrals no
  
          # TLS CA Certificate File
          TLSCACertFile   /usr/local/etc/ssl/ca.pem
  
          # TLS CA Certificate Directory
          TLSCACertDir    /etc/ssl/certs
  
          # Client Certificate and key
          # If TLS client authentication is required
          # TLSCertFile   /usr/local/etc/ssl/client-cert.pem
          # TLSKeyFile    /usr/local/etc/ssl/client-key.pem
  
          # Cipher Suite
          # The defaults are usually fine here
          # TLSCipherSuite        ALL:!ADH:@STRENGTH
  </LDAP>
  
  <Authorization>
          # Base DN
          BaseDN          "cn=users,cn=accounts,dc=<your_domain_components>"
  
          # User Search Filter
          SearchFilter    "(uid=%u)"
  
          # Require Group Membership
          RequireGroup    false
  
          # Add non-group members to a PF table (disabled)
          #PFTable        ips_vpn_users
  
          <Group>
                  BaseDN          "cn=groups,cn=accounts,dc=<your_domain_components>"
                  SearchFilter    "(|(cn=group1)(cn=group2)...(groupX))"
                  MemberAttribute uniqueMember
                  # Add group members to a PF table (disabled)
                  #PFTable        ips_vpn_eng
          </Group>
  </Authorization>
  ```
</details>  
4. Add a line below in /etc/openvpn/server.conf
```
plugin /usr/lib/openvpn/openvpn-auth-ldap.so /etc/openvpn/auth/auth-ldap.conf
```
5. Restart openvpn service and check the LDAP connection between the VPN server and LDAP server
```
sudo systemctl restart openvpn
sudo netstat -puant | grep 389
```
