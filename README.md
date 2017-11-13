# aci-ansible-plays
Basic repo containing Ansible plays to deliver a tenant config to the fabric, end to end.

To try to cover as much as possible I have included different vendors where possible

### Pre-requisites
- python >= 2.7.9
- ansible >= 2.4.0 (can use a venv if you feel brave)
- python f5sdk module

### Topology
Basic topology looks like this: 
```text
      Classical L2 Switching      +                 ACI Switching/Routing/Faffery
+---------------------------------+------------------------------------------------------------+

+----------------+        +----------------+        +-----------------+
|                |        |             p39+--------+p1               |XXXX
|g0  Internet  g1+--------+p16 Fortinet    |        |   Border Leaf   |    X
|                |        |             p40+---+  +-+p2    (101)      |     X
+----------------+        +----------------+   |  | +-----------------+      X
                                               |  |                   X       X+---------------+
                                             +----+                    X       |               |
                          +----------------+ | |    +-----------------+XXXXXXXX|    Spine 1    |
                          |            p2.1+-+ +----+p1               |  X    X|               |
                          |     F5 LTM     |        |   Border Leaf   |   X  X +---------------+
                          |            p2.2+--------+p2    (102)      |X   XX  X
                          +----------------+        +-----------------+ X  XX  X
                                                                         XX  XX
                                                                         XX  XX
                          +----------------+        +-----------------+ X  XX  X
                          |              p8+--------+1                |X   XX  X
                          |   Cisco ASA    |        |   System Leaf   |   X  X +---------------+
                          |     Core     p9+---+  +-+16    (103)      |  X    X|               |
                          +----------------+   |  | +-----------------+XXXXXXXX|    Spine 2    |
                                               |  |                    X       |               |
                                             +----+                   X       X+---------------+
                          +----------------+ | |    +-----------------+      X
                          |               0+-+ +----+1                |     X
                          |   ESXi Host    |        |   System Leaf   |    X
                          |               1+--------+16    (104)      |XXXX
                          +----------------+        +-----------------+
```
Port Map:

| Device                | Local Port | Remote Device  | Remote Port|  Role                                             |
| :-------------------: | :---------:| :------------: | :--------: | :-----------------------------------------------: | 
| Internet Router       | g0        | $ISP            | $port      | Transit BGP                                       |
| Internet Router       | g1        | Forti FW        | 16         | BGP Announced Range                               |
| Fortinet Perimeter FW | 16        | Internet Router | g1         | Firewall Outside                                  |
| Fortinet Perimeter FW | 39        | Border Leaf 101 | 1          | Firewall Inside LACP Member `aci-vpc`             |
| Fortinet Perimeter FW | 40        | Border Leaf 102 | 1          | Firewall Inside LACP Member `aci-vpc`             |
| F5 LTM                | 2.1       | Border Leaf 101 | 2          | F5 on a stick LACP Member `aci_vpc`               |
| F5 LTM                | 2.2       | Border Leaf 102 | 2          | F5 on a stick LACP Member `aci_vpc`               |
| ASA 55xx Core FW      | 8         | System Leaf 103 | 1          | Internal Firewall LACP Member `port-channel 1`    |
| ASA 55xx Core FW      | 8         | System Leaf 104 | 1          | Internal Firewall LACP Member `port-channel 1`    |
| ESXi Host             | 0         | System Leaf 103 | 16         | Internal Compute Resource MAC-Pinned Port-channel | 
| ESXi Host             | 1         | System Leaf 104 | 16         | Internal Compute Resource MAC-Pinned Port-channel |

---
## Basic Use
1) In the base of the repo validate the port map reflects your environment (use the port map for search and replace I guess?)
2) Run the `playme.yaml` wrapper with `tags=all`
3) Wait for the ~~sky to fall~~ build process to complete
4) ~~scream at your console~~ Celebrate

### Advanced Use
Tags:
- all: run the playbook end to end - skip nothing - do everything
- bindings: run just the sections that would be required when adding/changing/modifying a concrete device
- l3out: run just the sections that would be required when adding/changing/modifying an l3out
- epgs: run just the sections that would be required when adding/changing an EPG
- contracts: run just the sections that would be required when adding/changing/mapping a contract of filter

