# dns-privacy-setup
Documentation for the installation and configuration of Unbound and Pi-hole to enhance DNS privacy and security on your network.

## Prerequisites
- Linux-based system (Ubuntu/Debian)
- Root or sudo access
- Pi-hole and Unbound repositories already installed


## A. Installation
1. **Install Unbound**
   ```bash
   sudo apt install unbound -y
   ```
   If you are installing unbound from a package manager, it should install the root.hints file automatically with the dependency dns-root-data. The root hints will then be automatically updated by your package manager.
   
   >**Optional:** Optional: Download the current root hints file (the list of primary root servers which are serving the domain "." - the root domain). Update it roughly every six months. Note that this file changes infrequently. This is only necessary if you are not installing unbound from a package manager. If you do this optional step, you will need to uncomment the root-hints: configuration line in the suggested config file.
   ```bash
   wget https://www.internic.net/domain/named.root -qO- | sudo tee /var/lib/unbound/root.hints
   ```
   
2. **Install Pi-hole**
   ```bash
   curl -sSL https://install.pi-hole.net | bash
   ```

## B. Configuration
1. **Unbound Configuration**
   Configure `Unbound`
   Highlights:
   - Listen only for queries from the local Pi-hole installation (on port 5335)
   - Listen for both UDP and TCP requests
   - Verify DNSSEC signatures, discarding BOGUS domains
   - Apply a few security and privacy tricks

    `etc/unbound/unbound.conf.d/pi-hole.conf`:    
   > **Note:** You can find the `pihole.conf` file for Unbound configuration in the repository. You can [download or copy the file here](https://github.com/andreassebayang/dns-privacy-setup/blob/8ed06aeb1de6cdd51c0d4b90cd3e33ef2b0783c1/pi-hole.conf).

   - After editing the file don't forget to `ctrl + o` and  `ctrl  +  x` `enter`
   - Create log dir and file, set permissions:
     ```bash
     sudo mkdir -p /var/log/unbound
     sudo touch /var/log/unbound/unbound.log
     sudo chown unbound /var/log/unbound/unbound.log
     ```
   - Restart the service
     `sudo systemctl restart unbound `

3. **Pi-hole DNS Setup**
   - Configuring Pi-hole to use Unbound as the upstream DNS resolver.
   - Go to the http://pi.hole/admin/ or http://youripserver/admin
     ![image](https://github.com/user-attachments/assets/7da065a6-e908-4206-bb97-9e916b3cd404)
     
   - After that you go to the settings, find DNS tab and change the Custome 1 (IPv4) to be 127.0.0.1#5335
     > **Note:** Make sure the port number specified for Unbound matches the port you have configured in the `unbound.conf` or `pi-hole.conf` file. This is crucial for the DNS resolver to function correctly.
   - Change the Interface settings to the Respond only on interface eth0 and `SAVE`
     ![image](https://github.com/user-attachments/assets/69e91faa-db5c-444e-a811-682843e623c8)


## C. Testing
- How to verify your new DNS setup.
  
- test dig `dig pi-hole.net @127.0.0.1 -p 5335`
  
![image](https://github.com/user-attachments/assets/0a8ae868-4045-4b27-9660-efe9a1871060)

-  test dig `dig dnssec.works @127.0.0.1 -p 5335`
  
  ![image](https://github.com/user-attachments/assets/d6d6df91-e9d0-41d9-b6d5-4cc7cbeee0ef)
> **Note:** Status must `NOERROR` plus resolve an IP address.

## D. Troubleshooting
- Common solutions for issues that may arise during installation or configuration.
- check `unbound-checkconf1
- find log error `sudo journalctl -xe`  or `sudo journalctl -u unbound` or `sudo journalctl -u pihole-FTL`
- find conflict port `netstat -tuln` , `netstat -tulnp | grep :53` , `netstat -tulnp | grep :5335` , `netstat -tulnp | grep :8953`
- restart service `sudo systemctl restart unbound` or `sudo systemctl restart pihole-FTL`
- check log `sudo tail -f /var/log/pihi


## E. Always Check
- Update pihole `pihole -up`
- update pihole database for blocking `pihole -g`
- Check DNSSEC `sudo unbound-anchor`

## F. Monitor
- Check processiing unbound `sudo unbound-control stats`
- Check log `sudo  tail -f /var/log/unbound/unbound.log`
- Check processing pihole `pihole -c`
- Check log `sudo tail -f /var/log/pihole/pihole.log`
- Monitor status pihole `pihole status`

## References
- [Pi-hole GitHub Repository](https://github.com/pi-hole/pi-hole)
- [Unbound GitHub Repository](https://github.com/NLnetLabs/unbound)

- [Pi-hole Documentation](https://docs.pi-hole.net/)
- [Unbound Documentation](https://nlnetlabs.nl/projects/unbound/about/)

![Pi-hole Badge](https://img.shields.io/github/stars/pi-hole/pi-hole?style=social)
![Unbound Badge](https://img.shields.io/github/stars/NLnetLabs/unbound?style=social)


## Contributing
If you would like to contribute to this repository or improve the integration between Pi-hole and Unbound, feel free to fork the repository and open a pull request. We welcome any suggestions or improvements!

Additionally, you can contribute to the official Pi-hole and Unbound projects:
- [Contribute to Pi-hole](https://github.com/pi-hole/pi-hole/blob/master/CONTRIBUTING.md)
- [Contribute to Unbound](https://github.com/NLnetLabs/unbound/blob/master/CONTRIBUTING.md)
