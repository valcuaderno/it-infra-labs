# Week 09 – Enterprise Network Security with FortiGate

## 1. Project Overview

This project implements an enterprise network security environment using EVE-NG, FortiGate Firewall, Cisco Layer 2/Layer 3 switches, Kali Linux, and Linux endpoints.

The objective was to build an enterprise-style network, configure firewall policies, implement Network Address Translation (NAT), apply security profiles, and demonstrate Intrusion Prevention System (IPS) detection using a custom signature.

**Project Focus:** Network Security Engineering  
**Lab Platform:** EVE-NG Community  
**Firewall:** FortiGate VM64 – FortiOS 7.4.13  
**Switching:** Cisco IOL Layer 2 and Layer 3  
**Security Testing:** Kali Linux

---

## 2. Lab Objectives

- Design an enterprise network topology.
- Configure FortiGate interfaces and IP addressing.
- Implement static routing.
- Configure separate IT and HR network segments.
- Create firewall address objects and service groups.
- Implement outbound firewall policies.
- Configure NAT for internet connectivity.
- Apply SSL inspection and IPS security profiles.
- Create and test a custom IPS signature.
- Monitor firewall sessions and security events.
- Verify traffic processing and policy matching.

---

## 3. Lab Environment

| Component | Technology |
|---|---|
| Host OS | Windows 11 |
| RAM | 16 GB |
| Virtualization | VMware Workstation Pro |
| Network Emulator | EVE-NG Community |
| Firewall | FortiGate VM64 |
| Firewall Version | FortiOS 7.4.13 |
| Core Switch | Cisco IOL Layer 3 |
| Access Switch | Cisco IOL Layer 2 |
| Security Testing | Kali Linux |
| Linux Endpoint | TinyCore Linux |
| Packet Analysis | Wireshark |

---

## 4. Network Architecture

The lab uses a segmented enterprise network with FortiGate positioned between the internal core network and the external network.

```text
                 EXTERNAL NETWORK
                        |
                   [ WAN CLOUD ]
                        |
                    FORTIGATE
                   port1: WAN
                 192.168.85.134
                        |
                   port2: LAN
                  10.255.0.1/30
                        |
                 CISCO L3 CORE
                  10.255.0.2/30
                        |
                   TRUNK LINK
                        |
                CISCO L2 ACCESS
                   /         \
                  /           \
             VLAN 10         VLAN 20
                IT              HR
                 |               |
              IT-PC            HR-PC
          192.168.10.10    192.168.20.10
                 |
             KALI LINUX
          192.168.10.50
                 |
           TINYCORE LINUX
          192.168.10.11
```

*Note: Kali Linux and TinyCore are connected to VLAN 10. The diagram is a logical representation of the lab.*

---

## 5. IP Addressing Plan

| Device | Interface | IP Address | Purpose |
|---|---|---|---|
| FortiGate | port1 | 192.168.85.134/24 | WAN |
| FortiGate | port2 | 10.255.0.1/30 | Core transit |
| FortiGate | port3 | Reserved | Unused |
| Core Switch | Ethernet0/0 | 10.255.0.2/30 | Firewall transit |
| Core Switch | Ethernet0/1.10 | 192.168.10.1/24 | IT gateway |
| Core Switch | Ethernet0/1.20 | 192.168.20.1/24 | HR gateway |
| IT-PC | eth0 | 192.168.10.10/24 | IT endpoint |
| TinyCore | eth0 | 192.168.10.11/24 | Linux endpoint |
| Kali Linux | Network interface | 192.168.10.50/24 | Security testing |
| HR-PC | eth0 | 192.168.20.10/24 | HR endpoint |

### Default gateways

- IT VLAN: 192.168.10.1
- HR VLAN: 192.168.20.1
- FortiGate WAN gateway: 192.168.85.2

---

## 6. Routing Configuration

### Core Switch

Configured Layer 3 routing between IT and HR VLANs using 802.1Q subinterfaces.

| Subinterface | VLAN | Gateway |
|---|---|---|
| Ethernet0/1.10 | 10 | 192.168.10.1/24 |
| Ethernet0/1.20 | 20 | 192.168.20.1/24 |

Default route:

```cisco
ip route 0.0.0.0 0.0.0.0 10.255.0.1
```

### FortiGate Static Routes

```shell
config router static
    edit 1
        set dst 192.168.10.0 255.255.255.0
        set gateway 10.255.0.2
        set device "port2"
    next
    edit 2
        set dst 192.168.20.0 255.255.255.0
        set gateway 10.255.0.2
        set device "port2"
    next
end
```

### Routing Verification

Command:

```shell
get router info routing-table all
```

Verified:
- Default route through 192.168.85.2 via port1.
- Connected transit network 10.255.0.0/30.
- IT network 192.168.10.0/24 through 10.255.0.2.
- HR network 192.168.20.0/24 through 10.255.0.2.
- Connected WAN network 192.168.85.0/24.

**Result: Successful.**

---

## 7. Firewall Policies

Two outbound firewall policies were implemented to control internet access for the IT and HR networks.

### Policy 1 – IT to Internet

| Setting | Configuration |
|---|---|
| Policy Name | Policy1-IT_TO_INTERNET |
| Incoming Interface | port2 |
| Outgoing Interface | port1 |
| Source | IT_NETWORK |
| Destination | all |
| Service | WEB_DNS_SERVICES |
| Action | ACCEPT |
| NAT | Enabled |
| Logging | All |
| UTM | Enabled |
| SSL Inspection | certificate-inspection |
| IPS Sensor | IT_IPS_LAB |

### Policy 2 – HR to Internet

| Setting | Configuration |
|---|---|
| Policy Name | Policy 2 – HR_TO_INTERNET |
| Incoming Interface | port2 |
| Outgoing Interface | port1 |
| Source | HR_NETWORK |
| Destination | all |
| Service | WEB_DNS_SERVICES |
| Action | ACCEPT |
| NAT | Enabled |
| Logging | All |
| UTM | Enabled |
| SSL Inspection | certificate-inspection |
| IPS Sensor | IT_IPS_LAB |

Both policies were verified through the FortiGate CLI.

---

## 8. Address Objects and Service Groups

Created address objects:

| Object | Network |
|---|---|
| IT_NETWORK | 192.168.10.0/24 |
| HR_NETWORK | 192.168.20.0/24 |

Created service group:

**WEB_DNS_SERVICES**

Included:
- DNS
- HTTP
- HTTPS

The service group was applied to both outbound firewall policies.

---

## 9. Network Address Translation (NAT)

Configured source NAT on both outbound firewall policies.

Example:

```text
Original Source:
192.168.20.10

Translated Source:
192.168.85.134

Destination:
8.8.8.8
```

### NAT Verification

Command:

```shell
diagnose sys session list
```

Verified:
- IT traffic matched Policy ID 1.
- HR traffic matched Policy ID 2.
- Internal source addresses were translated to 192.168.85.134.
- Traffic received responses from external destinations.

**Result: Successful.**

---

## 10. Intrusion Prevention System (IPS)

Configured an IPS sensor named:

`IT_IPS_LAB`

The sensor initially used severity-based inspection for medium, high, and critical severity entries.

A custom IPS signature was created to demonstrate signature-based traffic detection.

### Custom Signature

- Name: `IPS-LAB-HTTP-TEST`
- Attack ID: `7117`
- Detection Pattern: `IPS-LAB-TEST-2026`
- Protocol: HTTP

Configuration:

```shell
config ips custom
    edit "IPS-LAB-HTTP-TEST"
        set signature "F-SBID( --attack_id 7117; --name \"IPS-LAB-HTTP-TEST\"; --service HTTP; --flow from_client; --pattern \"IPS-LAB-TEST-2026\"; )"
        set action block
    next
end
```

### Adding the Signature to the IPS Sensor

```shell
config ips sensor
    edit "IT_IPS_LAB"
        config entries
            edit 2
                set rule 7117
                set status enable
                set action block
            next
        end
    next
end
```

### IPS Testing

Generated HTTP traffic from Kali Linux:

```bash
curl -v "http://example.com/IPS-LAB-TEST-2026"
```

The FortiGate Intrusion Prevention Security Events log successfully recorded the custom signature.

### Observed Results

| Test | Result |
|---|---|
| Custom signature creation | Successful |
| Signature added to IPS sensor | Successful |
| Kali HTTP test traffic | Generated |
| Custom signature detection | Successful |
| IPS security event logging | Successful |
| Blocking enforcement | Inconclusive |

**Important:** Although the custom signature was configured with a block action, the HTTP request continued to return HTTP 200 and the event showed `detected`. Blocking enforcement was not verified.

---

## 11. FortiGate Session Monitoring

Used:

```shell
diagnose sys session list
```

Verified HR traffic:

```text
192.168.20.10 -> 8.8.8.8:53
```

NAT translation:

```text
192.168.85.134
```

Firewall policy:

```text
policy_id=2
```

IPS inspection path indicator:

```text
redir-to-ips
```

This confirmed that HR traffic matched Policy 2 and was redirected to the IPS inspection path.

---

## 12. Connectivity and Validation Tests

| Test | Result |
|---|---|
| IT to default gateway | Passed |
| HR to default gateway | Passed |
| Core switch to FortiGate transit | Passed |
| IT to 8.8.8.8 | Passed |
| HR to 8.8.8.8 | Passed |
| IT DNS resolution | Passed |
| HR DNS resolution | Passed |
| IT outbound firewall policy | Passed |
| HR outbound firewall policy | Passed |
| IT source NAT | Passed |
| HR source NAT | Passed |
| Custom IPS signature detection | Passed |
| Custom IPS blocking | Inconclusive |

---

## 13. FortiGuard and IPS Limitations

During testing, FortiGuard connectivity was unavailable.

The FortiGate IPS engine was installed, but the attack definitions reported an update date of December 1, 2015.

Limitations identified:
- FortiGuard definition updates were unavailable.
- Existing IPS signatures could still be used.
- Custom IPS signature detection worked.
- Current threat signature coverage could not be guaranteed.
- IPS blocking behavior was not successfully validated.

This lab demonstrates IPS configuration and custom signature detection, not production-grade threat protection.

---

## 14. Technical Skills Demonstrated

### Networking
- Enterprise network topology design
- VLAN segmentation
- 802.1Q trunking
- Inter-VLAN routing
- Static routing
- Default routing
- IPv4 addressing and subnetting

### FortiGate Security
- Interface configuration
- Firewall policies
- Address objects
- Service groups
- Source NAT
- SSL certificate inspection
- IPS sensor configuration
- Custom IPS signatures
- Security event monitoring
- Session table investigation

### Security Testing and Troubleshooting
- Kali Linux traffic generation
- HTTP request inspection
- IPS event validation
- Firewall policy verification
- NAT troubleshooting
- FortiGuard connectivity investigation

---

## 15. Lessons Learned

1. Firewall policies must match the correct incoming and outgoing interfaces.
2. Static routes are essential for communication between the firewall and internal networks.
3. NAT allows internal networks to access external destinations using the firewall WAN address.
4. Security profiles must be attached to firewall policies to inspect matching traffic.
5. Custom IPS signatures can provide useful detection tests without relying on new FortiGuard signatures.
6. An IPS detection event does not automatically prove traffic was blocked.
7. Session-table inspection helps identify the firewall policy and NAT behavior applied to traffic.
8. Configuration validation should be supported by actual traffic tests and security logs.

---

## 16. Project Conclusion

Successfully implemented an enterprise-style network security lab using FortiGate and Cisco switching inside EVE-NG.

The project demonstrated network segmentation, firewall policy configuration, routing, NAT, security profile integration, and custom IPS signature detection.

The project also provided practical experience investigating FortiGuard update limitations, validating firewall sessions, and distinguishing between IPS detection and prevention.

This project forms part of my hands-on development toward a **Network Security Engineer / Hybrid Network and System Security Engineer** role.

---

## 17. Evidence and Screenshots

Recommended repository structure:

```text
Week-09-FortiGate-Enterprise-Security/
│
├── README.md
│
└── screenshots/
    ├── 01-enterprise-network-topology.png
    ├── 02-fortigate-interfaces.png
    ├── 03-routing-table.png
    ├── 04-firewall-policy-it.png
    ├── 05-firewall-policy-hr.png
    ├── 06-nat-session-table.png
    ├── 07-custom-ips-signature.png
    ├── 08-ips-detection-event.png
    └── 09-hr-policy-session-verification.png
```

Only include actual screenshots captured from the lab.

**Project Status:** Core configuration and IPS detection completed. IPS blocking validation remains inconclusive.

**Learning Path:** 26-Week Cybersecurity Engineering Academy  
**Week:** 09 – FortiGate Enterprise Network & Security
