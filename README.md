# AWS-site-to-site-vpn-wireguard-1
# Site-to-Site VPN with WireGuard on AWS

A hybrid cloud networking project that establishes an encrypted site-to-site 
VPN tunnel between a simulated on-premises network and an AWS VPC using WireGuard.

# Architecture

[On-Prem-Server 10.0.x.x]
        |
        | (local network traffic)
        |
[On-Prem-VPN-Server 10.0.1.7] ══ Encrypted WireGuard Tunnel ══ [Cloud-VPN-Server 13.49.23.184]
  Public IP: 13.60.67.225                (over public internet)       |
        |                                                              |
Simulated Office Network                                    AWS VPC (172.16.0.0/16)
(10.0.0.0/16)                                               eu-north-1 (Stockholm)

## Infrastructure

| Instance | Role | Network |

| Cloud-VPN-Server | AWS-side VPN gateway | 172.16.0.0/16 (AWS VPC) |
| On-Prem-VPN-Server | Office-side VPN gateway | 10.0.0.0/16 (Simulated on-prem) |
| On-Prem-Server | End-user workstation | 10.0.0.0/16 (Simulated on-prem) |

All instances: t3.micro, Ubuntu, eu-north-1 (Stockholm)

## Key Technical Decisions

### WireGuard as the VPN Protocol
- Built into the Linux kernel (5.6+) — no extra drivers needed
- ~4,000 lines of code vs OpenVPN's 70,000+ — smaller attack surface
- Uses modern cryptography: ChaCha20, Curve25519, Poly1305
- Significantly faster than IPSec or OpenVPN
- Self-managed alternative to AWS Site-to-Site VPN 

### Public/Private Key Authentication
- Each server generates its own key pair locally
- Private keys never leave their host server
- Public keys are exchanged between peers
- Asymmetric encryption — intercepted traffic cannot be decrypted without the private key

### Source/Destination Check Disabled
- AWS drops packets not addressed to/from the instance's own IP by default
- VPN gateways forward traffic on behalf of other machines
- Disabling this check enables the instances to act as routers/gateways

### Security Group Configuration
| Rule | Protocol | Port | Source | Purpose |

| WireGuard | UDP | 51820 | 0.0.0.0/0 | VPN tunnel traffic |
| SSH | TCP | 22 | 0.0.0.0/0 | Server management |
| ICMP | ICMP | All | 172.16.0.0/16 | Cross-network ping/testing |

## What This Demonstrates

- Hybrid cloud networking architecture
- VPN tunnel design and implementation
- AWS VPC, subnet, and security group configuration
- Linux networking and routing concepts
- Encrypted communication between private networks
- Cost-conscious infrastructure design

## Verified Results

- WireGuard handshake confirmed between both VPN gateways
- 0% packet loss across the encrypted tunnel
- Bidirectional traffic flow verified (604B received, 820B sent)
- Persistent keepalive every 25 seconds maintaining tunnel stability

## Technologies Used

- AWS EC2 (t3.micro, Ubuntu)
- AWS VPC, Subnets, Security Groups
- WireGuard VPN
- Linux networking (ip route, systemctl)
- eu-north-1 (Stockholm) region
