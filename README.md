# Enterprise-Campus-LAN-Network-Topology

A simulated enterprise campus network built in CPT, implementing a full three-tier hierarchical design (core/distribution/access) with redundant Layer 3 uplinks, per-VLAN load balancing across dual distribution switches, first-hop redundancy, and port-level hardening.

This project was built as part of my hands-on CCNA 200-301 preparation and networking portfolio.

## Topology

<img width="787" height="579" alt="image" src="https://github.com/user-attachments/assets/7727fe52-d96c-4042-95ec-b34513179f50" />


| Tier | Devices                          |                       Role                                   |
|------|----------------------------------|--------------------------------------------------------------|
| Core | 1x Layer 3 switch (`CORE-SW`)    | Single routed exit point to the server subnet; OSPF backbone |
| Distribution | 2x Layer 3 switches      | Inter-VLAN routing, HSRP gateway redundancy, STP root per VLAN, peer link
               |(`DIST1-SW`, `DIST2-SW`)  |
| Access | 4x Layer 2 switches (`ASW1`–`ASW4`) | End-user connectivity, dual-homed trunk uplinks, port security |
| Endpoints | 1x server, 16x PCs | 8 VLANs, 2 hosts each |

Every access switch is dual-homed — one trunk to each distribution switch — so no single switch or link failure isolates an end user.

## Design highlights

- **Per-VLAN load balancing, not just a passive standby link.** VLANs 10/30/50/70 are forwarded via `DIST1-SW` (STP root + HSRP active); VLANs 20/40/60/80 are forwarded via `DIST2-SW`. Both uplinks off every access switch carry live production traffic simultaneously instead of one sitting idle as a cold backup.
- **STP and Layer 3 gateway are intentionally aligned per VLAN.** The switch that's STP root for a VLAN is also the HSRP active router for it, so traffic never takes an indirect path across the distribution peer link.
- **OSPF single-area backbone** across core and both distribution switches for automatic failover if a distribution switch or the peer link goes down.
- **HSRP first-hop redundancy** on every VLAN gateway — sub-second failover if either distribution switch fails, with priorities set to match the STP design above.
- **Least-privilege trunking.** Each access switch's uplinks only carry the two VLANs it actually hosts (not all eight) — this keeps STP/broadcast scope tight and limits VLAN-hopping exposure. Full-VLAN trunks are reserved for links that actually need them (distribution downlinks and the distribution peer link).
- **Port security on every access port** — one sticky MAC per port, `restrict` violation mode, so a port doesn't go dark on a false positive but unauthorized devices are still blocked and logged.
- **Native VLAN hardening** — trunk native VLAN moved off VLAN 1 to an unused VLAN 999 to reduce VLAN-hopping risk.

---

## IP addressing

### Point-to-point links (routed)

| Link | Subnet | Endpoint A | Endpoint B |
|---|---|---|---|
| CORE ↔ DIST1 | 192.168.1.64/30 | CORE Gi0/1 = .65 | DIST1 Gi0/1 = .66 |
| CORE ↔ DIST2 | 192.168.1.68/30 | CORE Gi0/2 = .69 | DIST2 Gi0/1 = .70 |
| CORE ↔ Server | 192.168.1.72/30 | CORE Fa0/1 = .73 | Server = .74 |
| DIST1 ↔ DIST2 (peer) | 192.168.1.76/30 | DIST1 Gi0/2 = .77 | DIST2 Gi0/2 = .78 |

### VLANs

| VLAN | Subnet | DIST1 SVI | DIST2 SVI | HSRP VIP (gateway) | STP root / HSRP active |
|---|---|---|---|---|---|
| 10 | 192.168.1.0/29 | .1 | .2 | .3 | DIST1 |
| 20 | 192.168.1.8/29 | .9 | .10 | .11 | DIST2 |
| 30 | 192.168.1.16/29 | .17 | .18 | .19 | DIST1 |
| 40 | 192.168.1.24/29 | .25 | .26 | .27 | DIST2 |
| 50 | 192.168.1.32/29 | .33 | .34 | .35 | DIST1 |
| 60 | 192.168.1.40/29 | .41 | .42 | .43 | DIST2 |
| 70 | 192.168.1.48/29 | .49 | .50 | .51 | DIST1 |
| 80 | 192.168.1.56/29 | .57 | .58 | .59 | DIST2 |

Full per-host addressing (PCs, exact ports, HSRP priorities) is in [`docs/build-guide.md`](docs/build-guide.md).

## Verification performed

- [x] `show ip ospf neighbour` on `CORE-SW` shows full adjacency with both distribution switches
- [x] `show standby brief` confirms `DIST1-SW` active for VLANs 10/30/50/70, standby for 20/40/60/80 (and the mirror on `DIST2-SW`)
- [x] `show spanning-tree vlan <id>` on each access switch confirms the uplink alternates between forwarding/blocking per VLAN as designed
- [x] `show interfaces trunk` confirms each access switch only advertises its own two VLANs
- [x] End-to-end ping: every PC → its own gateway, the server, and PCs on other VLANs/switches (routing + inter-switch STP/trunking all confirmed)
- [x] Failure test: distribution peer link and an access uplink shut down individually to confirm reconvergence via OSPF/HSRP/STP without manual intervention

---

## Skills demonstrated

·`VLAN segmentation` 
· `802.1Q trunking` 
· `inter-VLAN routing (SVI)` 
· `OSPF` · `HSRP` 
· `Rapid-PVST+ with manual root/priority tuning` 
· `per-VLAN load balancing` · `port security` 
· `structured IP subnetting (VLSM)` 
· `enterprise 3-tier hierarchical design`


## Author

*Ayomide Oyekunle*
