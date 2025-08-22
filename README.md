# pfSense Firewall Lab

## Objectives
- Deploy a **pfSense firewall** in VirtualBox with two interfaces:  
  - **WAN (Bridged)** → simulates the external/internet-facing side.  
  - **LAN (Internal Network)** → simulates a protected internal network.  
- Place **Kali Linux** on the WAN side to act as an **external attacker**.  
- Place **Ubuntu Desktop** on the LAN side as the **victim host**.  
- Simulate a **Denial of Service (DoS)** attack using `hping3` from Kali.  
- Capture and analyze the traffic using **Wireshark**.  
- Create and test **firewall rules in pfSense** to mitigate the attack.  

## Lab Topology
          +---------------------+
          |  Kali Linux (Attacker) 
          |  WAN: 192.168.2.x
          +---------------------+
                     |
          (WAN - Bridged Adapter)
                     |
             +------------------+
             |   pfSense        |
             |   WAN: 192.168.2.25
             |   LAN: 192.168.1.1
             +------------------+
                     |
        (LAN - Internal Network)
                     |
          +---------------------+
          |  Ubuntu Desktop (Victim)
          |  LAN: 192.168.1.100
          +---------------------+


---

## Reaching pfSense from the WAN
By default, pfSense blocks GUI access from the WAN for security. To allow management from WAN during the lab:

1. Temporarily disable pfSense firewall from the shell:  
   ```bash
   Option: 8 -> to access the shell
   pfctl -d
   ```
2. Log into pfSense from a browser using the WAN IP (ex: https://192.168.2.25).
  Default credentials:
    ```bash
    Username: admin
    Password: pfsense
    ```

3. Add a firewall rule to permanently allow WAN GUI access:
- Go to Firewall ▸ Rules ▸ WAN ▸ Add
- Action: Pass
- Protocol: TCP
- Source: your host/home network (192.168.2.0/24)
- Destination: pfSense WAN address (192.168.2.25)
- Description: "Allow GUI access from home network"
- Save and Apply.

## Configure LAN DHCP
- Go to Services ▸ DHCP Server.
- Enable DHCP on the LAN.
- Settings:
  - Range: 192.168.1.100 – 192.168.1.200
  - DNS Server: 1.1.1.1 or 8.8.8.8
- Save and Apply.
</br>
👉 Now the Ubuntu Desktop (LAN side) will automatically receive an IP (e.g., 192.168.1.100).


## Launching the DoS (hping3)

From Kali Linux, simulate a flood attack against the Ubuntu host through pfSense:
```bash
sudo hping3 -S --flood -V -p 80 192.168.1.100
```
- -S → TCP SYN flood
- --flood → send packets as fast as possible
- -p 80 → target port 80 (web)
- 192.168.1.100 → target victim (Ubuntu)
</br>
👉 Run Wireshark on Ubuntu (or pfSense) to observe the flood.


## Blocking the Attack & Verifying
- In pfSense, go to Firewall ▸ Rules ▸ WAN.
- Add a rule to Block traffic from Kali’s IP (192.168.2.26):
  - Action: Block
  - Protocol: Any
  - Source: 192.168.2.26
  - Destination: 192.168.1.100
  - Save and Apply
- Monitor logs:
  - Status ▸ System Logs ▸ Firewall
  - Confirm that traffic from Kali is being blocked.
- Re-run the DoS attack → it should now be blocked, and Ubuntu should not see the flood.


## Lessons Learned / Security Concepts Demonstrated
- Segregation of Networks (WAN vs LAN):
  - Demonstrated how pfSense separates external (untrusted) and internal (trusted) networks.

- Default Deny Principle:
  - By default, WAN → LAN traffic is denied, showing the importance of least privilege access.

- Firewall Rule Management:
  - Showed how specific rules can allow controlled access (NAT/port forwarding) or block malicious hosts.

- Intrusion Simulation:
  - Used hping3 to replicate a DoS/SYN flood and confirmed traffic in Wireshark.

- Detection and Logging:
  - Observed firewall logs to identify attacker traffic and validate security controls.

- Mitigation Techniques:
  - Applied IP-based blocking at the firewall to stop attack traffic effectively.

- Hands-On Threat Modeling:
  - Simulated the attack chain from reconnaissance (ping) to exploitation (DoS) and mitigation (rules).


## Reference
[Youtube](https://www.youtube.com/watch?v=-yRvfbElT7M)
