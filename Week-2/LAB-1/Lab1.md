# CSYS 4483-01 / CSYS 6683-01  
# Network Defense  
# Lab 1: Packet Crafting with Scapy  

**Group Members:**  
- Kismat Kunwar  
- Tyler Joseph  

**Date of Submission:**  
- September 9, 2025  

## 1. Objective
The goal of this lab is to craft TCP SYN packets that contain data and observe how the target system responds. This demonstrates a weakness in TCP design as described in RFC 793.  

## 2. Setup
- Target: Ubuntu VM running Apache web server. (Ip Addres-10.0.0.8 ) 
- Attacker: Kali Linux VM.  (10.0.0.101)
- Tools installed: Apache server, Wireshark.  
- Configuration: Apache configured to listen on port 80

## 3. Implementation
- On the Ubuntu VM:  
  - Installed Apache and created a test page in `/var/www/index.html`.  
  - Configured Apache to listen on the VM interface at port 80.  
  - Started Wireshark and captured traffic on the active interface.

- On the Kali VM:  
  - Wrote Python script `script1.py` using Scapy.  
   - Set destination IP to the Ubuntu VM.  
  - Added data payload inside the SYN packet.  
  - Ran the script with execution permissions

    ```python

    #!/bin/env python
    from scapy.all import *
    import time

    dst_ip = "10.0.0.8"
    sport = 5555
    payload = 'GET / HTTP/1.1\r\nHost: 10.0.0.8\r\n\r\n'

    # SYN with payload (abnormal)
    syn = IP(dst=dst_ip)/TCP(dport=80, sport=sport, flags='S')/payload
    syn.display()

    # Send SYN and get SYN-ACK
    syn_ack = sr1(syn, timeout=2)
    syn_ack.display()

    # ACK back to server
    ack = IP(dst=dst_ip)/TCP(dport=80, sport=sport,
                            flags='A',
                            seq=syn_ack.ack,
                            ack=syn_ack.seq + 1)
    send(ack)

    time.sleep(2)

    # Close with RST
    rst = IP(dst=dst_ip)/TCP(dport=80, sport=sport,
                            flags='R',
                            seq=syn_ack.ack + 1,
                            ack=syn_ack.seq + 1)
    send(rst)

    ```
 

## 4. Observations
- Wireshark shows TCP SYN packets that contain data in the payload.First packet, 10.0.0.101:5555 to 10.0.0.8:80, TCP SYN with payload. TCP Segment Len 34. Next Seq 35. Wireshark decodes the payload as HTTP, request line GET / HTTP/1.1.
**Snapshot 1:** TCP SYN with data packet visible in Wireshark.  

- The server responded , 10.0.0.8:80 to 10.0.0.101:5555  with a TCP SYN-ACK packet. The SYN-ACK acknowledged the connection request but ignored the payload.  

**Snapshot 2:** Response packet (SYN-ACK) from Ubuntu VM. 

- After receiving the SYN-ACK, the Kali VM responded with a RST packet since no valid session was established.  
- Script follow up, TCP ACK from 10.0.0.101:5555 to 10.0.0.8:80. Flags ACK. Seq 1. Ack server_seq+1. Len 0.
- No HTTP transaction appears in the captured flow.


**Snapshot 2:**  Remaining Connection

 
## 5. Analysis

Ubuntu followed RFC 793 behavior. RFC 793 allows data in a SYN. Most stacks ignore such data during the handshake. The server acknowledged only the Initial Sequence Number + 1. Not Initial Sequence Number + 1 + payload length. Example, SYN payload 34 bytes. Server ACK 1 relative, not 35. Result, the TCP layer discarded the SYN payload. Apache logged no request.

Kali sent RST due to missing kernel socket state for 10.0.0.101:5555 to 10.0.0.8:80. Raw packet injection created no SYN-SENT state. The script ACK arrived after the reset and had no effect.

Security takeaway. Data inside a SYN receives uneven handling across stacks. Flag SYN segments with payload during monitoring. Keep services configured to ignore such data.  

