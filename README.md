# BGP_OSPF
BGP

Part 1: BGP Neighbor Finite State Machine (FSM)
 [Idle]  --->  [Connect]  <--->  [Active]
                 |                  |
                 v                  v
            [OpenSent]  ----------> +
                 |
                 v
           [OpenConfirm]
                 |
                 v
           [Established]

1. Idle
* What happens: This is the initial state. BGP refuses all incoming connections and is waiting for a start event (e.g., neighbor configured, interface comes up, or manual clear/reset).
* Normal transition: Once the process starts, BGP allocates resources, starts the ConnectRetry timer, and transitions to Connect.
2. Connect
* What happens: BGP initiates a TCP connection (3-way handshake) to the peer on TCP Port 179.
* Normal transition:
    * If the TCP 3-way handshake succeeds, BGP sends an OPEN message and transitions to OpenSent.
    * If the TCP connection fails or the ConnectRetry timer expires, BGP transitions to Active.
3. Active
* What happens: The router failed to initiate the TCP connection and is actively listening/trying to establish the TCP connection on port 179.
* Normal transition:
    * If the TCP connection completes while in Active state, BGP sends an OPEN message and moves to OpenSent.
    * If the ConnectRetry timer expires again without success, BGP resets the timer and falls back to Connect.
4. OpenSent
* What happens: BGP has sent its OPEN message and is waiting to receive an OPEN message from the peer.
* Validation: When the remote OPEN frame arrives, BGP checks for parameter matches: BGP Version (4), Autonomous System (AS) number, BGP Router ID, Hold Time negotiation, and Capabilities (e.g., IPv4/IPv6 Unnumbered, MP-BGP).
* Normal transition: If parameters are valid, BGP sends a KEEPALIVE message and transitions to OpenConfirm. If there is a mismatch, BGP sends a NOTIFICATION message and resets to Idle.
5. OpenConfirm
* What happens: BGP is waiting for a KEEPALIVE or NOTIFICATION message from the peer to complete negotiation.
* Normal transition: Upon receiving the remote peer's KEEPALIVE, BGP transitions to Established.
6. Established
* What happens: The BGP neighbor adjacency is fully formed. Peers begin exchanging UPDATE messages (prefix routes) and periodic KEEPALIVE messages to maintain the session.

Part 2: Troubleshooting Why BGP Neighbors Won't Come Up
When a BGP session gets stuck, the specific state it is stuck in points directly to the underlying root cause.
Scenario A: Stuck in Idle
* Root Causes:
    1. Interface Down: The local interface pointing to the peer is down or administratively shutdown.
    2. No Route to Peer: The routing table has no valid route to reach the peer's IP address.
    3. Neighbor Soft-Reset / Shutdown: The neighbor is administratively disabled (neighbor <IP> shutdown).
    4. 
Scenario B: Stuck in Active or Flapping between Connect & Active
This indicates a Layer 1 to Layer 4 TCP connectivity failure. The router is trying to reach the peer on TCP port 179 but receives no response (or receives a TCP RESET).
* Root Causes:
    1. IP Reachability / Routing Issue: No route to the peer, or asymmetric routing where return traffic is dropped.
    2. Firewall / ACL / CoPP Blocking Port 179: A firewall, Access Control List, or Control Plane Policing (CoPP) policy is dropping TCP port 179 packets.
    3. Incorrect Neighbor IP: Typo in the neighbor's IP address configuration.
    4. Source IP / Update-Source Mismatch: The peer expects BGP packets from a Loopback IP, but the local switch is sending them using a physical interface IP (missing update-source loopback or equivalent).
    5. eBGP Multihop Missing: eBGP peers connected across multiple hops have an initial IP TTL of 1. If they are not directly connected, packets get dropped due to TTL expiry unless ebgp-multihop is configured.
Scenario C: Stuck in OpenSent or OpenConfirm
This indicates TCP connectivity was established, but the BGP protocol-level handshake failed during parameter negotiation.
* Root Causes:
    1. AS Number Mismatch: Configured remote-as on one router does not match the local AS configured on the peer.
    2. Router ID Collision: Both BGP routers have identical Router IDs (e.g., both defaulted to 1.1.1.1).
    3. BGP Version / Capability Mismatch: Unmatching address family capabilities (e.g., IPv4 vs IPv6 Unnumbered, MP-BGP features).
    4. Hold Time Mismatch: Unacceptable Hold Time negotiation values.
    5. Authentication Password Mismatch: MD5 key / BGP password mismatch causing invalid TCP MD5 checksums.
Scenario D: Reaches Established then Intermittently Flaps to Idle
This indicates that the session forms, but control-plane traffic is interrupted or delayed during operation.
* Root Causes:
    1. Hold-Timer Expiry (Keepalive Drops): BGP keepalives fail to arrive within the hold-time window (typically 30 seconds or 180 seconds).
    2. Control Plane / CPU Starvation (CoPP Issue): Heavy host traffic, ARP storms, masscan events, or DLF (Destination Lookup Failure) punts overload the CPU core, dropping BGP keepalive packets before the zebra/FRR routing daemon processes them.
    3. MTU Mismatch / Packet Fragmentation: BGP session opens (small TCP packets succeed), but large UPDATE messages carrying many routes exceed the path MTU and get dropped if fragmentation is disabled or path MTU discovery (PMTUD) fails.
    4. Link Flapping & Unstable Uplinks: Transient link state changes cause underlying FDB/ARP flushes, causing control plane instability.


10-Celestica
11-Edgecore
12-Wistron with dell 5232
14-Dell
16-Nvidia SN serues& Ufispace -s6301
17-Cisco
