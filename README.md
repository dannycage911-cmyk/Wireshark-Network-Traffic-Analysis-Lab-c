# Wireshark-Network-Traffic-Analysis-Lab-c
Danny cage Wireshark Network Traffic Analysis Lab

🔎 Wireshark Network Traffic Analysis Lab

📌 Project Overview

This project documents a practical Wireshark network traffic analysis lab designed to develop fundamental packet-analysis and defensive cybersecurity skills.

The lab uses a deliberately isolated Python HTTP server running on 127.0.0.1:8081 to generate controlled network traffic. Wireshark is then used to capture, filter, inspect, and analyze that traffic.

The exercises demonstrate how network analysts can identify:

* TCP connection establishment and termination
* HTTP requests and responses
* Cleartext credentials transmitted over HTTP
* DNS queries and responses
* Network traffic statistics
* Files transferred over HTTP
* File integrity using SHA-256 hashes
* TCP port-scanning behavior

⚠️ Important: All traffic in this project is generated against my own local test environment. The HTTP server is intentionally insecure and is used only for educational and defensive analysis.

⸻

🎯 Project Objectives

The primary objectives of this lab were to:

1. Learn how to capture network traffic using Wireshark.
2. Understand how TCP establishes connections using the three-way handshake.
3. Use Wireshark display filters to isolate relevant packets.
4. Demonstrate why unencrypted HTTP is insecure.
5. Analyze DNS queries and responses.
6. Use Wireshark statistics to understand network conversations.
7. Extract files transferred over HTTP.
8. Verify extracted files using SHA-256 hashing.
9. Identify the characteristics of a TCP port scan.
10. Develop practical skills in network monitoring and defensive analysis.

⸻

🛠️ Tools and Environment

Software Used

Tool	Purpose
Wireshark	Packet capture and network traffic analysis
Python 3	Local HTTP test server
Web Browser	Generate HTTP traffic
cURL	Generate HTTP requests from the command line
Nmap	Generate a controlled TCP port scan
SHA-256	Verify file integrity

Network Environment

The main HTTP test server runs locally:

127.0.0.1:8081

The server intentionally uses plain HTTP so that the contents of the communication can be observed in Wireshark.

The lab therefore provides a controlled environment for demonstrating how information can be exposed when encryption is not used.

⸻

🧪 Lab Setup

1. Start the Test Server

The lab uses a Python server named:

lab_server.py

The server listens only on:

http://127.0.0.1:8081

Start the server with:

Linux/macOS

python3 lab_server.py

Windows

python lab_server.py

or:

py lab_server.py

Expected output:

Serving on http://127.0.0.1:8081

I then opened the address in a browser to confirm that the test login page was available.

⸻

📡 Exercise 1 — First Packet Capture

Objective

The first exercise focused on creating a basic Wireshark packet capture.

Procedure

1. Open Wireshark.
2. Select the loopback interface.
3. Start a new capture.
4. Visit:

http://127.0.0.1:8081/

5. Reload the page several times.
6. Stop the capture.
7. Save the capture as:

lab1.pcapng

What I Observed

The capture contained TCP traffic associated with the local HTTP server.

Wireshark provided three important analysis panels:

* Packet List — individual captured packets
* Packet Details — protocol and field information
* Packet Bytes — raw packet data


Figure 1 — Initial Wireshark Packet Capture


⸻

🔍 Exercise 2 — Wireshark Display Filters

Objective

Display filters were used to reduce a large packet capture to only the traffic relevant to the investigation.

Filters Used

tcp.port == 8081

Shows TCP traffic involving port 8081.

http

Displays HTTP traffic.

http.request

Displays HTTP requests.

http.request.method == "GET"

Displays HTTP GET requests.

http.response.code == 200

Displays successful HTTP responses.

http && !(http.request)

Displays HTTP traffic excluding requests.

frame contains "Lab Login"

Searches packet contents for the text Lab Login.

Key Lesson

Wireshark display filters do not delete packets. They only control which packets are currently displayed.

⸻

🤝 Exercise 3 — TCP Three-Way Handshake

Objective

The purpose of this exercise was to identify the TCP three-way handshake:

SYN → SYN/ACK → ACK

Filter

tcp.flags.syn == 1

The connection begins with a SYN packet sent by the client.

The server responds with:

SYN + ACK

The client then completes the handshake with:

ACK

TCP Handshake

Client                         Server
  |                              |
  | -------- SYN --------------> |
  |                              |
  | <------ SYN + ACK --------- |
  |                              |
  | -------- ACK --------------> |
  |                              |
  |       Connection Ready       |

Packet Characteristics

SYN

SYN = 1
ACK = 0

SYN-ACK

SYN = 1
ACK = 1

ACK

SYN = 0
ACK = 1

Stream Analysis

The following filter can be used to inspect an entire TCP conversation:

tcp.stream eq 0

This allows the analyst to view the handshake, HTTP traffic, and connection termination within the same TCP stream.


Figure 2 — TCP Three-Way Handshake

<img width="1265" height="71" alt="Screenshot 2026-10-05 at 12 14 23 AM" src="https://github.com/user-attachments/assets/afd537a9-d503-411f-acdd-e42473b8b6b8" />

Label the following packets:
1. SYN
2. SYN-ACK
3. ACK

⸻

🔐 Exercise 4 — HTTP Cleartext Credential Demonstration

Objective

This exercise demonstrates the security risk associated with transmitting credentials over unencrypted HTTP.

A deliberately fake login was submitted to the local test server.

Lab Credentials

Username: student
Password: LabPass123

⚠️ These credentials are lab-only test credentials and should never be reused for real accounts.

The lab specifically uses this controlled demonstration to show how credentials can appear directly inside captured HTTP traffic.

Filter Used

http.request.method == "POST"

This identifies the HTTP POST request containing the form submission.

Following the HTTP Stream

The packet can then be opened using:

Right-click → Follow → HTTP Stream

The captured request can expose form data such as:

username=student&password=LabPass123

Security Implication

The demonstration shows why sensitive information should not be transmitted over plain HTTP.

Because HTTP does not provide encryption, an attacker capable of observing the network traffic could potentially read application data transmitted in cleartext.



Figure 3 — Follow HTTP Stream

<img width="818" height="470" alt="Screenshot 2026-10-05 at 12 17 29 AM" src="https://github.com/user-attachments/assets/2f792d1b-d2cb-46f3-b4dc-e5b55fefe25a" />

NOTE:
Credentials shown are intentionally generated lab credentials.

⸻

🌐 Exercise 5 — DNS Traffic Analysis

Objective

This exercise analyzed DNS queries and responses using the active network interface.

DNS traffic was generated using:

nslookup example.com

and:

nslookup wireshark.org

Filter

dns

This displays DNS traffic.

Additional Filters

Only queries:

dns.flags.response == 0

Only responses:

dns.flags.response == 1

Search for a specific domain:

dns.qry.name contains "example"

Analysis

The DNS response can be expanded in Wireshark to identify the resolved IP address.

The destination address of the DNS request can also be inspected to identify the DNS server contacted by the system.

The lab notes that DNS commonly uses UDP port 53.

📸 Screenshot

Figure 4 — DNS Query and Response

[INSERT SCREENSHOT HERE]

⸻

📊 Exercise 6 — Wireshark Statistics

Objective

The statistics tools were used to understand the volume and structure of the generated traffic.

Thirty HTTP requests were generated against the local server.

Linux/macOS

for i in $(seq 1 30); do curl -s http://127.0.0.1:8081/ > /dev/null; done

Windows PowerShell

1..30 | ForEach-Object { curl.exe -s http://127.0.0.1:8081/ | Out-Null }

Statistics Examined

Protocol Hierarchy

Statistics → Protocol Hierarchy

This provides a breakdown of protocols observed in the capture.

Conversations

Statistics → Conversations → TCP

This allows individual TCP conversations to be examined.

Endpoints

Statistics → Endpoints → IPv4

The lab environment should show:

127.0.0.1

as the local endpoint.

I/O Graphs

Statistics → I/O Graphs

The traffic should appear as a short burst corresponding to the generated requests.

⸻

📁 Exercise 7 — HTTP Object Extraction

screenshot

<img width="632" height="113" alt="Screenshot 2026-10-05 at 1 16 02 AM" src="https://github.com/user-attachments/assets/84329343-f90d-4abe-8119-250ccb6e256a" />


Objective

This exercise demonstrated how a file transferred over HTTP can be extracted from a packet capture.

The server provides:

/logo.png

The original file was downloaded separately:

curl -s -o original_logo.png http://127.0.0.1:8081/logo.png

Wireshark Procedure

Navigate to:

File → Export Objects → HTTP

Locate:

logo.png

and export it as:

exported_logo.png

Integrity Verification

The original and extracted files were compared using SHA-256.

Linux

sha256sum original_logo.png exported_logo.png

macOS

shasum -a 256 original_logo.png exported_logo.png

Windows PowerShell

Get-FileHash original_logo.png, exported_logo.png

SHA-256 Results

File	SHA-256
original_logo.png	[INSERT ACTUAL HASH]
exported_logo.png	[INSERT ACTUAL HASH]

Expected Result

If the extracted file is identical to the original, both SHA-256 values should match.

original_logo.png
        ↓
SHA-256
        ↓
[HASH]
exported_logo.png
        ↓
SHA-256
        ↓
[HASH]
Result: MATCH

The actual hashes must be taken from my own lab execution rather than fabricated.

⸻

🚨 Exercise 8 — Recognizing a TCP Port Scan

Objective

The final exercise demonstrated how a TCP port scan appears from a defensive network-monitoring perspective.

The scan was restricted to my own local machine:

127.0.0.1

Nmap Command

nmap -sT -p 8070-8090 127.0.0.1

This scans ports 8070 through 8090.

SYN Filter

tcp.flags.syn == 1 && tcp.flags.ack == 0

This isolates initial TCP connection attempts.

What Indicates a Scan?

A port scan produces a recognizable pattern:

Source → Port 8070
Source → Port 8071
Source → Port 8072
Source → Port 8073
...
Source → Port 8090

The packets occur rapidly and target multiple consecutive ports.

RST Filter

tcp.flags.reset == 1

Closed ports can respond with TCP reset packets.

The lab server’s port:

8081

should respond differently because the service is listening there.

Port-Specific Filter

tcp.port == 8081 && tcp.flags.syn == 1

This can be used to focus on the connection involving the open test-server port.

📸 Screenshot

Figure 5 — TCP SYN Packets Indicating a Port Scan

<img width="1172" height="262" alt="Screenshot 2026-10-05 at 1 42 39 AM" src="https://github.com/user-attachments/assets/d4b9416a-04ea-4a1f-a73a-a5f6ae8ece75" />

The screenshot should show multiple SYN packets
targeting different ports in a short period.

⸻

🧰 Wireshark Filters Reference

Purpose	Wireshark Filter
TCP port 8081	tcp.port == 8081
HTTP traffic	http
HTTP requests	http.request
GET requests	http.request.method == "GET"
HTTP 200 responses	http.response.code == 200
HTTP responses	http && !(http.request)
Search packet contents	frame contains "Lab Login"
SYN packets	tcp.flags.syn == 1
Initial SYN only	tcp.flags.syn == 1 && tcp.flags.ack == 0
TCP stream	tcp.stream eq 0
HTTP POST	http.request.method == "POST"
DNS traffic	dns
DNS queries	dns.flags.response == 0
DNS responses	dns.flags.response == 1
DNS domain search	dns.qry.name contains "example"
TCP resets	tcp.flags.reset == 1
Port 8081 SYN traffic	tcp.port == 8081 && tcp.flags.syn == 1

⸻


The following screenshots should be added to this repository after completing the lab.

Screenshot 1 — TCP Three-Way Handshake

[INSERT IMAGE]
Description:
Wireshark capture showing the SYN, SYN-ACK, and ACK packets
used to establish the TCP connection.

Screenshot 2 — HTTP Stream

[INSERT IMAGE]
Description:
Follow HTTP Stream showing the intentionally generated
lab credentials in cleartext.

Screenshot 3 — SHA-256 Verification

[INSERT IMAGE]
Description:
Terminal/PowerShell output showing matching SHA-256 hashes
for original_logo.png and exported_logo.png.

Screenshot 4 — Port Scan

[INSERT IMAGE]
Description:
Wireshark showing multiple SYN packets directed toward
different ports during the controlled Nmap scan.

⸻

📋 Results

Exercise	Result
Exercise 1	Successfully captured local HTTP traffic
Exercise 2	Successfully applied Wireshark display filters
Exercise 3	Identified TCP SYN, SYN-ACK, and ACK
Exercise 4	Demonstrated cleartext HTTP form data
Exercise 5	Captured and analyzed DNS queries/responses
Exercise 6	Examined protocol hierarchy, conversations, endpoints, and I/O
Exercise 7	Extracted HTTP object and performed SHA-256 comparison
Exercise 8	Identified the packet pattern associated with a TCP port scan

Key Evidence

TCP Handshake:
[INSERT CAPTURE RESULT]

HTTP Stream:
[INSERT SCREENSHOT/OBSERVATION]

Original SHA-256:
[INSERT HASH]

Exported SHA-256:
[INSERT HASH]

Hash Match:
[YES / NO]

Port Scan Observation:
[INSERT OBSERVATION]

⸻

🔐 Security Observations

1. Unencrypted HTTP Exposes Application Data

The HTTP demonstration showed that information submitted through an unencrypted connection can be visible inside a packet capture.

This reinforces the importance of using encrypted protocols such as HTTPS for sensitive communications.

2. Packet Analysis Provides Visibility

Wireshark provides detailed visibility into network communications, including:

* Source and destination addresses
* Ports
* Protocols
* TCP flags
* Application-layer information
* Transferred objects

3. TCP Flags Can Reveal Network Behavior

TCP flags are useful indicators for defensive monitoring.

For example:

SYN
SYN-ACK
ACK

normally represents connection establishment, while a rapid series of SYN packets targeting multiple ports can indicate scanning activity.

4. File Integrity Can Be Verified Cryptographically

SHA-256 provides a practical way to determine whether an extracted file is identical to the original.

Matching hashes indicate that the files produced the same SHA-256 digest.

5. Network Monitoring Helps Detect Suspicious Activity

A defender can use packet captures and filters to identify unusual behaviors such as:

* Rapid connection attempts
* Port scanning
* Unexpected protocols
* Cleartext credentials
* Suspicious DNS activity
* Unexpected file transfers

⸻

🧠 Lessons Learned

This lab provided practical experience with several important cybersecurity concepts.

Network Traffic Analysis

I learned how individual packets can be examined to understand what is happening on a network.

TCP Communication

I learned how the TCP three-way handshake establishes a connection and how TCP flags can be used during packet analysis.

Protocol Filtering

I learned how Wireshark filters make it easier to isolate specific traffic from a larger capture.

Network Security

The HTTP credential demonstration showed why sensitive information should not be transmitted over unencrypted protocols.

File Integrity

I learned how SHA-256 can be used to compare an original file with a file extracted from network traffic.

Threat Detection

The port-scan exercise demonstrated how repeated connection attempts against sequential ports can provide an indication of reconnaissance activity.

⸻

⚖️ Ethical and Legal Scope

This project was performed in a controlled environment for educational and cybersecurity-training purposes.

All network activity was intentionally generated against:

127.0.0.1

which refers to the local machine.

The port scan was also restricted to the local host.

No unauthorized systems, networks, accounts, or third-party infrastructure were targeted.

Security testing should only be performed against systems for which explicit authorization has been obtained.

⸻

📂 Suggested GitHub Repository Structure

wireshark-network-analysis-lab/
│
├── README.md
│
├── captures/
│   ├── lab1.pcapng
│   ├── lab4.pcapng
│   ├── lab6.pcapng
│   ├── lab7.pcapng
│   └── lab8.pcapng
│
├── screenshots/
│   ├── tcp-handshake.png
│   ├── http-stream.png
│   ├── sha256-verification.png
│   └── port-scan.png
│
├── server/
│   └── lab_server.py
│
├── evidence/
│   ├── original_logo.png
│   └── exported_logo.png
│
└── LICENSE

Privacy note: Do not upload captures containing real passwords, personal information, private DNS lookups, or other sensitive data. The lab instructions specifically recommend deleting captures containing lab credentials or personal lookups when they are no longer needed.

⸻

🚀 How to Reproduce the Lab

Clone the repository:

git clone https://github.com/YOUR-USERNAME/wireshark-network-analysis-lab.git

Enter the project directory:

cd wireshark-network-analysis-lab

Start the local server:

python3 lab_server.py

Open:

http://127.0.0.1:8081

Start Wireshark and select the appropriate loopback interface.

Perform the exercises described in this documentation and save the resulting captures/screenshots.

⸻

🏁 Conclusion

This Wireshark lab provided practical experience in network traffic capture, packet inspection, protocol analysis, and defensive security monitoring.

The exercises demonstrated how TCP connections are established, how HTTP traffic can expose information when encryption is absent, how DNS requests can be analyzed, how network statistics can reveal communication patterns, how transferred files can be recovered and verified, and how port scans can be recognized from packet-level behavior.

Overall, the project strengthened my understanding of how a security analyst can use network packet analysis to investigate communications and identify potentially suspicious activity.

⸻

👤 Author

Daniel 

Cybersecurity Student / Security Analyst

⸻

⭐ Project Skills Demonstrated

Wireshark
Network Traffic Analysis
Packet Capture
TCP/IP Analysis
HTTP Analysis
DNS Analysis
Network Security
Incident Detection
File Integrity Verification
SHA-256
Nmap
Defensive Security
Cybersecurity Fundamentals
