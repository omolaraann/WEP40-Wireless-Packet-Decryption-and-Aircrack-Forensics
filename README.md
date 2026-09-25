# 🛡️ WEP40 Wireless Packet Decryption & Aircrack Forensics

## 🔎 Lab Overview

This repository documents the forensic examination of a historical **WEP40 wireless network capture** conducted as part of the SBT-DF203 Digital Forensics practical laboratory.

The exercise focuses on the preservation, validation, analysis, offline WEP key recovery, decryption, network reconstruction, HTTP traffic examination, and recovery of application-layer objects from a supplied historical wireless packet capture.

The investigation was performed against an **offline, authorised historical capture**. No live wireless networks were targeted, accessed, or attacked.

The primary objectives were to demonstrate how weak WEP encryption can be investigated from captured traffic and to show how decrypted wireless traffic can subsequently be analysed using standard network-forensic techniques.

---

## 🎯 Investigation Objectives

The laboratory investigation covered the following areas:

* 🗃️ Preservation of the supplied historical evidence
* 🔐 SHA-256 integrity verification
* 📦 Creation of a controlled working copy
* 🗜️ Decompression and file-format validation
* 📡 Identification of IEEE 802.11 traffic
* 🧩 Classification of wireless frame types and subtypes
* 🔒 Identification of WEP-protected frames
* 🔁 Analysis of WEP Initialization Vector reuse
* 🔑 Offline WEP40 key recovery using Aircrack-ng
* 🔓 Offline wireless traffic decryption using Airdecap-ng
* 🧾 Hashing of the decrypted evidence derivative
* 🌐 Reconstruction of IP, MAC, TCP and UDP communications
* 🔍 HTTP request and response analysis
* 🕸️ Examination of historical PhishMe web activity
* 📁 HTTP object extraction
* 🧪 File-type identification of recovered objects
* #️⃣ SHA-256 hashing of recovered objects
* 📝 Documentation of forensic findings and limitations

---

## ⚖️ Authorisation & Evidence Handling

This investigation was performed exclusively against the historical wireless capture supplied for the laboratory exercise.

The recovered WEP key was used only to decrypt the supplied offline capture. No attempt was made to recover credentials, access a live wireless network, bypass security controls on an active system, or interact with any external service identified within the historical traffic.

The original compressed evidence was preserved separately from the working copy. Analysis was performed against the working evidence and subsequently generated derivatives.

This separation helps maintain a clear distinction between:

**Original Evidence → Working Copy → Decompressed Capture → Decrypted Derivative → Recovered Objects**

---

## 📁 Repository Structure

```text
SBT-DF203-Lab9/
│
├── 📂 evidence/
│   └── file.xz
│
├── 📂 working/
│   ├── file_working.xz
│   ├── file_working
│   └── file_working-dec
│
├── 📂 exported/
│   └── Recovered HTTP objects
│
├── 📂 reports/
│   ├── file_xz_sha256.txt
│   ├── working_hashes.txt
│   ├── decrypted_capture_sha256.txt
│   ├── recovered_object_hashes.txt
│   ├── recovered_object_file_types.txt
│   └── recovered_object_inventory.txt
│
├── 📂 screenshots/
│   └── Laboratory evidence screenshots
│
└── 📂 scripts/
    └── Supporting analysis scripts
```

---

## 🧰 Tools Used

The investigation utilised standard Linux and network-forensic utilities.

| Tool                      | Purpose                                       |
| ------------------------- | --------------------------------------------- |
| 🐧 Kali Linux             | Forensic analysis environment                 |
| 🔍 Wireshark              | Packet inspection and graphical analysis      |
| 🖥️ TShark                | Command-line packet analysis                  |
| 🔑 Aircrack-ng            | Offline WEP key recovery                      |
| 🔓 Airdecap-ng            | Offline WEP packet decryption                 |
| #️⃣ SHA-256               | Evidence and object integrity verification    |
| 📄 `file`                 | File-signature and object-type identification |
| 📦 `unxz`                 | Decompression of supplied evidence            |
| 🌐 `wget`                 | Retrieval of the supplied historical capture  |
| 📁 `find` / `du`          | Recovered-object inventory                    |
| 🧩 `foremost` / `binwalk` | Available for additional forensic examination |

---

## 📥 Evidence Acquisition

The supplied historical capture was downloaded into the dedicated evidence directory.

```bash
wget -O evidence/file.xz \
'https://raw.githubusercontent.com/ctfs/write-ups-2015/master/codegate-ctf-2015/programming/good-crypto/file.xz'
```

The acquisition initially encountered a transfer/read error. The download was subsequently resumed and completed successfully.

The final evidence file size was:

```text
12,837,660 bytes
```

The completed evidence was retained under:

```text
evidence/file.xz
```

---

## 🔐 Original Evidence Integrity

A SHA-256 hash was generated immediately after acquisition:

```bash
sha256sum evidence/file.xz | tee reports/file_xz_sha256.txt
```

The resulting SHA-256 value was:

```text
dd54144caef34f228bfb4b87a9101ca7173969376f7ed357bd068061d4f4b8d6
```

This hash serves as the primary integrity reference for the supplied compressed evidence.

---

## 📋 Working Copy

To avoid performing analysis directly against the preserved evidence file, a working copy was created:

```bash
cp --preserve=timestamps evidence/file.xz working/file_working.xz
```

The working copy retained the original evidence timestamps.

Its SHA-256 hash was subsequently compared with the original evidence.

Both produced:

```text
dd54144caef34f228bfb4b87a9101ca7173969376f7ed357bd068061d4f4b8d6
```

This confirmed that the working compressed copy was byte-for-byte identical to the acquired evidence.

---

## 🗜️ Evidence Decompression

The working compressed file was decompressed while retaining the original `.xz` file:

```bash
unxz -k working/file_working.xz
```

The resulting capture was validated using:

```bash
file working/file_working
```

The capture was identified as:

```text
pcap capture file, microsecond ts (little-endian)
version 2.4
802.11
capture length 65535
```

This confirmed that the decompressed working file was a packet capture containing IEEE 802.11 wireless traffic.

---

## 📡 Initial Wireless Traffic Analysis

The protocol hierarchy was examined using TShark:

```bash
tshark -r working/file_working -q -z io,phs
```

The capture contained:

```text
45,169 frames
13,511,274 bytes
```

The protocol hierarchy identified WLAN traffic and approximately:

```text
15,713 data frames
```

The capture therefore contained a substantial volume of wireless traffic suitable for further WEP analysis.

---

## 🧩 IEEE 802.11 Frame Analysis

Wireless frame types and subtypes were enumerated using:

```bash
tshark -r working/file_working -Y 'wlan.fc.type' \
-T fields -e wlan.fc.type -e wlan.fc.subtype \
| sort | uniq -c
```

The analysis identified management, control and data traffic.

Significant observations included:

* 📡 Beacon frames
* 🔎 Probe responses
* 🤝 Authentication and association activity
* 📶 RTS/CTS traffic
* ✅ ACK traffic
* 📦 Data frames
* 📦 QoS data frames
* 🔄 Null and QoS-null frames
* 🧱 Block acknowledgement traffic

The presence of these different frame categories demonstrates that the capture represents an actual wireless network communication environment rather than an isolated collection of encrypted packets.

---

## 🔒 WEP-Protected Traffic

The WEP initialization-vector field was identified as:

```text
wlan.wep.iv
```

Protected frames were examined using:

```bash
tshark -r working/file_working \
-Y 'wlan.fc.protected == 1' \
-T fields \
-e frame.number \
-e frame.time_relative \
-e wlan.sa \
-e wlan.da \
-e wlan.bssid \
-e wlan.wep.iv
```

A total of:

```text
15,713
```

protected frames were identified.

The protected traffic was associated with the BSSID:

```text
00:26:66:55:97:d6
```

The capture therefore provided sufficient WEP-protected traffic for further cryptographic analysis.

---

## 🔁 WEP Initialization Vector Reuse

WEP Initialization Vectors were examined for repeated values:

```bash
tshark -r working/file_working \
-Y 'wlan.fc.protected == 1' \
-T fields -e wlan.wep.iv \
| sort | uniq -c | sort -nr | head -20
```

Repeated IV values were identified.

Examples included:

```text
0x42e8eb    7 occurrences
0x28d3eb    7 occurrences
0x95dfeb    6 occurrences
0x9415ed    5 occurrences
0x86e5eb    5 occurrences
```

The repeated IVs demonstrate that the wireless network reused initialization vectors within the captured WEP traffic.

IV reuse is a significant forensic observation because WEP uses a relatively small 24-bit IV space and combines the IV with the shared secret during RC4 key generation. Reuse of IVs can therefore provide attackers with statistical information that contributes to recovery of the WEP key.

The repeated IV values themselves do not constitute the recovered key. They are evidence of the weakness in the captured WEP implementation.

---

## 🔢 Protected Traffic Distribution

The protected-frame count was independently verified:

```bash
tshark -r working/file_working \
-Y 'wlan.fc.protected == 1' \
-T fields -e frame.number \
| wc -l
```

Result:

```text
15713
```

BSSID distribution was also examined:

```bash
tshark -r working/file_working \
-Y 'wlan.fc.protected == 1' \
-T fields -e wlan.bssid \
| sort | uniq -c | sort -nr
```

The result showed:

```text
15713  00:26:66:55:97:d6
```

Thus, all identified protected frames were associated with the same wireless BSSID.

---

## 🔑 Offline WEP40 Key Recovery

Aircrack-ng was executed against the supplied historical capture:

```bash
aircrack-ng working/file_working
```

Aircrack-ng identified:

```text
BSSID: 00:26:66:55:97:D6
ESSID: cgnetwork
Encryption: WEP
IVs: 15477
```

The tool successfully recovered the WEP40 key:

```text
A4:3D:F6:F3:74
```

Aircrack-ng reported:

```text
KEY FOUND! [ A4:3D:F6:F3:74 ]
Decrypted correctly: 100%
```

The key recovery was performed entirely offline against the historical packet capture.

---

## 🔓 Offline WEP Decryption

The recovered WEP key was supplied to Airdecap-ng:

```bash
airdecap-ng -w A4:3D:F6:F3:74 working/file_working
```

Airdecap-ng reported:

```text
Total number of stations seen           10
Total number of packets read         45169
Total number of WEP data packets     15477
Total number of WPA data packets         0
Number of plaintext data packets         0
Number of decrypted WEP  packets     15477
Number of corrupted WEP  packets         0
```

The result demonstrates successful offline decryption of all:

```text
15,477 WEP data packets
```

with:

```text
0 corrupted WEP packets
```

This provided a validated decrypted derivative for subsequent network and application-layer forensic analysis.

---

## 🧾 Decrypted Capture Integrity

Airdecap-ng generated:

```text
working/file_working-dec
```

The resulting file was identified as an Ethernet packet capture:

```text
pcap capture file
version 2.4
Ethernet
capture length 65535
```

A SHA-256 hash was generated:

```bash
sha256sum working/file_working-dec \
| tee reports/decrypted_capture_sha256.txt
```

Hash:

```text
167c91994c269777f9048227deb89882caf3cf3c763977f2059604f9a6a40b04
```

This hash provides an integrity reference for the decrypted evidence derivative used during subsequent analysis.

---

## 🌐 Decrypted Network Protocol Analysis

The decrypted capture was analysed using:

```bash
tshark -r working/file_working-dec -q -z io,phs
```

The decrypted traffic contained:

```text
15,477 Ethernet frames
15,361 IPv4 frames
56 IPv6 frames
15,195 TCP frames
159 UDP frames
60 ARP frames
7 DHCP frames
81 DNS frames
76 mDNS frames
31 SSDP frames
7 IGMP frames
20 ICMPv6 frames
139 TLS frames
278 HTTP frames
```

The HTTP traffic also contained multiple application-layer object types, including:

* 🖼️ PNG
* 🖼️ GIF
* 🖼️ JPEG/JFIF
* 📜 JavaScript
* 🎨 CSS
* 📄 XML
* 🧾 JSON
* 📝 Text content

This demonstrated that successful WEP decryption exposed substantial application-layer traffic for forensic examination.

---

## 🖥️ Endpoint Analysis

IPv4 communication was analysed using:

```bash
tshark -r working/file_working-dec \
-Y 'ip' \
-T fields \
-e ip.src \
-e ip.dst \
| sort | uniq -c | sort -nr
```

The primary internal endpoint identified was:

```text
192.168.0.15
```

Significant external communications included:

```text
198.90.20.111
199.27.79.193
173.194.127.141
173.194.127.55
```

The highest-volume communications involved:

```text
192.168.0.15 ↔ 198.90.20.111
192.168.0.15 ↔ 199.27.79.193
```

These communications accounted for substantial portions of the decrypted TCP traffic.

---

## 🖧 MAC Address Correlation

Ethernet communication was analysed to identify dominant MAC-address relationships.

The principal communication pair was:

```text
00:26:66:55:97:d4
↔
f0:f6:1c:68:96:7c
```

ARP analysis subsequently correlated:

```text
192.168.0.1  → 00:26:66:55:97:d4
192.168.0.15 → f0:f6:1c:68:96:7c
192.168.0.9  → 48:5b:39:2a:c2:7a
```

An important distinction was maintained between the wireless BSSID:

```text
00:26:66:55:97:d6
```

and the gateway MAC address identified through ARP:

```text
00:26:66:55:97:d4
```

These addresses were not treated as interchangeable.

---

## 🔗 TCP Conversation Analysis

TCP conversations were examined using:

```bash
tshark -r working/file_working-dec -q -z conv,tcp
```

High-volume HTTP conversations included communications between:

```text
192.168.0.15:51007
↔
198.90.20.111:80
```

and:

```text
192.168.0.15:50959
↔
199.27.79.193:80
```

Several HTTP conversations transferred more than one megabyte of data.

HTTPS traffic was also identified, including communications over TCP/443.

A local TCP conversation involving:

```text
192.168.0.15:51036
↔
192.168.0.9:5000
```

was also observed.

The TCP conversation inventory provided the foundation for subsequent HTTP object recovery.

---

## 📡 UDP Conversation Analysis

UDP traffic included:

* 🔎 DNS
* 📣 mDNS
* 📺 SSDP
* 🌐 DHCP
* 📢 Other multicast/broadcast traffic

Examples included:

```text
192.168.0.1:2048
↔
239.255.255.250:1900
```

and:

```text
192.168.0.15:5353
↔
224.0.0.251:5353
```

IPv6 mDNS communication was also identified.

The presence of these protocols provided additional context regarding local service discovery and network configuration within the historical capture.

---

## 🌍 HTTP Traffic Analysis

HTTP requests and responses were extracted using:

```bash
tshark -r working/file_working-dec \
-Y 'http.request || http.response' \
-T fields \
-e frame.number \
-e ip.src \
-e ip.dst \
-e tcp.srcport \
-e tcp.dstport \
-e http.request.method \
-e http.host \
-e http.request.uri \
-e http.response.code \
-e http.content_type
```

The traffic showed historical web browsing activity involving multiple websites and supporting services.

Examples included:

```text
m.reddit.com
imgur.com
i.imgur.com
google-related services
advertising services
analytics services
phishme.com
```

The capture therefore contained sufficient application-layer information to reconstruct portions of the historical browsing activity.

---

## 🕸️ PhishMe Web Activity

A targeted analysis was performed for:

```text
phishme.com
```

The capture contained the request:

```text
GET /decoding-zeus-disguised-as-an-rtf-file/
```

from:

```text
192.168.0.15
```

to:

```text
198.90.20.111:80
```

The page subsequently generated requests for numerous supporting resources, including:

* CSS files
* JavaScript files
* PNG images
* analytics resources
* tracking resources
* WordPress-related resources

Multiple responses returned:

```text
HTTP 200
```

indicating successful retrieval of many requested resources.

The traffic therefore demonstrates historical browsing activity involving the PhishMe page.

However, the page title and URL alone do **not** establish that a ZeuS executable or RTF document was downloaded.

Such a conclusion would require direct evidence of the corresponding transferred object.

---

## 📦 HTTP Object Recovery

HTTP objects were exported from the decrypted capture using TShark:

```bash
tshark -r working/file_working-dec --export-objects http,exported
```

The recovered objects were stored under:

```text
exported/
```

The extracted material included:

* 🖼️ PNG images
* 🖼️ GIF images
* 🎨 CSS stylesheets
* 📜 JavaScript
* 🔤 Web fonts
* 📄 HTML
* 📊 Analytics responses
* 🛰️ Tracking resources
* ⚪ Empty HTTP responses

Duplicate objects were retained using filenames containing suffixes such as:

```text
(1)
(2)
(3)
```

This prevented later extracted objects from overwriting earlier files.

---

## 🧪 Recovered Object File-Type Identification

The recovered objects were examined using:

```bash
file exported/* | tee reports/recovered_object_file_types.txt
```

The results identified numerous ordinary web-resource formats, including:

```text
HTML document
JavaScript source
GIF image data
PNG image data
ASCII text
Web Open Font Format
CSS
empty
```

Several filenames contain references to:

```text
Decoding ZeuS Disguised as an .RTF File
```

However, those objects were identified as ASCII/analytics-related content rather than an RTF document.

No conclusion was therefore drawn solely from the filename or URL.

This distinction is important in forensic analysis because filenames, URLs and page titles describe application-layer context but do not necessarily establish the actual content transferred over the network.

---

## #️⃣ Recovered Object Hashing

SHA-256 hashes were generated for the recovered HTTP objects:

```bash
sha256sum exported/* | tee reports/recovered_object_hashes.txt
```

The resulting hash inventory provides integrity references for the recovered objects.

Identical hashes were observed among several duplicate filenames.

For example, multiple copies of the same extracted resource produced identical SHA-256 values.

This indicates that the duplicate files are byte-for-byte identical rather than different versions of the resource.

Zero-byte extracted objects also produced the standard SHA-256 empty-file value:

```text
e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
```

These objects were retained as part of the extraction record but do not contain substantive recovered data.

---

## 🔬 Key Forensic Findings

### 🔐 WEP Security Weakness

The historical wireless network used WEP40 encryption.

The capture contained significant quantities of WEP-protected traffic and repeated IV values.

The recovered key:

```text
A4:3D:F6:F3:74
```

was successfully validated through complete offline decryption.

### 🔓 Successful Decryption

Airdecap-ng successfully decrypted:

```text
15,477 WEP data packets
```

with:

```text
0 corrupted WEP packets
```

This confirms that the recovered key was valid for the captured WEP traffic.

### 🖥️ Primary Internal Endpoint

The principal internal IPv4 endpoint identified in the decrypted traffic was:

```text
192.168.0.15
```

This endpoint generated significant external HTTP traffic.

### 🌐 Application-Layer Visibility

Once WEP protection was removed, the capture exposed:

* HTTP
* DNS
* DHCP
* mDNS
* SSDP
* TLS
* ARP
* TCP
* UDP

This demonstrates the extent to which weak wireless encryption can expose higher-layer network activity.

### 📁 Object Recovery

HTTP object extraction successfully recovered a collection of web resources from the decrypted capture.

The recovered material predominantly consisted of ordinary webpage components rather than a clearly identified executable or RTF document.

### 🕵️ PhishMe Activity

The capture contains historical browsing activity involving the PhishMe page:

```text
/decoding-zeus-disguised-as-an-rtf-file/
```

The associated resources were successfully recovered from the HTTP traffic.

The page title provides contextual information about the subject of the webpage but is not, by itself, proof that malware was downloaded.

---

## ⚠️ Evidence Limitations

The investigation is based entirely on the supplied historical packet capture.

The following limitations should therefore be considered:

* 📦 Only traffic present within the supplied capture can be examined.
* 🕒 The evidence represents a historical snapshot rather than the complete activity of the endpoint.
* 🔒 HTTPS-encrypted content cannot necessarily be reconstructed without the required session-level evidence or keys.
* 📄 HTTP object extraction only recovers objects actually represented in the captured HTTP traffic.
* 🧩 The presence of a webpage discussing malware does not establish that malware was downloaded.
* 🧪 File-type identification is based on the recovered object content and signatures available to the `file` utility.
* 🗂️ Duplicate exported objects may represent repeated retrievals of the same resource.
* ⚪ Empty extracted objects represent HTTP transactions without recoverable response bodies.

These limitations prevent conclusions from being extended beyond what the packet evidence directly supports.

---

## 🛡️ Modern Wireless Security Controls

The forensic findings demonstrate why WEP should not be used in modern wireless environments.

Appropriate modern controls include:

* 🔐 WPA2-AES or WPA3-based wireless security
* 🚫 Complete removal of WEP
* 🔑 Strong, unique wireless authentication credentials
* 🏢 WPA2/WPA3-Enterprise where appropriate
* 🎫 802.1X authentication for enterprise environments
* 👁️ Wireless intrusion detection and prevention
* 📡 Continuous monitoring of rogue access points
* 🔄 Regular credential rotation
* 🧩 Network segmentation
* 🛡️ Endpoint security controls
* 🔍 Continuous monitoring of wireless infrastructure
* 📋 Formal wireless security configuration standards

The key lesson is that wireless encryption must provide confidentiality and resistance to practical cryptographic attacks; legacy WEP does not meet modern security requirements.

---

## 🧾 Evidence Chain Summary

The investigation followed the following evidence workflow:

```text
📥 Historical Capture
        ↓
🔐 SHA-256 Preservation Hash
        ↓
📋 Verified Working Copy
        ↓
🗜️ Decompression
        ↓
📡 802.11 / WEP Analysis
        ↓
🔁 IV Reuse Identification
        ↓
🔑 Offline WEP40 Key Recovery
        ↓
🔓 Offline Decryption
        ↓
🧾 Decrypted Capture Hash
        ↓
🌐 Network / Protocol Analysis
        ↓
🔍 HTTP Investigation
        ↓
📦 HTTP Object Extraction
        ↓
🧪 File-Type Identification
        ↓
#️⃣ Object SHA-256 Hashing
        ↓
📝 Forensic Findings
```

---

## 🏁 Conclusion

The laboratory investigation successfully demonstrated the forensic examination of a historical WEP40 wireless capture.

The supplied evidence was preserved and hashed before analysis, a verified working copy was created, and the compressed capture was successfully decompressed and validated as an IEEE 802.11 packet capture.

Analysis identified substantial WEP-protected traffic associated with the wireless BSSID:

```text
00:26:66:55:97:d6
```

Repeated WEP Initialization Vectors were observed within the protected traffic. Aircrack-ng subsequently recovered the WEP40 key:

```text
A4:3D:F6:F3:74
```

Airdecap-ng validated the recovered key by successfully decrypting all 15,477 captured WEP data packets without corrupted WEP packets.

The decrypted capture provided visibility into IP, MAC, TCP, UDP, DNS, DHCP, mDNS, ARP, HTTP and other application-layer communications. The principal internal endpoint identified was:

```text
192.168.0.15
```

HTTP analysis revealed historical browsing activity involving several external services, including a PhishMe webpage concerning ZeuS and an RTF file. HTTP object extraction recovered numerous webpage resources, which were subsequently classified and hashed.

The recovered-object analysis demonstrates the importance of distinguishing **contextual references within URLs, page titles and analytics data from direct evidence of transferred files**. Based on the recovered file-type evidence examined, the presence of the PhishMe page does not by itself establish that a ZeuS malware sample or RTF document was downloaded.

Overall, the exercise demonstrates the forensic consequences of legacy WEP encryption: once the weak wireless protection is defeated, captured traffic can expose network communications and application-layer content that would otherwise have remained protected.

---

## 📚 Evidence & Reports

Supporting forensic artefacts generated during the investigation are maintained within the repository, including:

```text
📄 Original evidence SHA-256
📄 Working-copy hashes
📄 Decrypted capture SHA-256
📄 Recovered-object SHA-256 inventory
📄 Recovered-object file-type inventory
📸 Analysis screenshots
📦 Recovered HTTP objects
```

All analysis should be interpreted together with the corresponding packet-capture evidence and command output rather than from individual filenames or isolated observations.

---

## 🛡️ Final Forensic Statement

**The investigation was conducted against an authorised historical WEP capture for educational and forensic-analysis purposes. No live wireless network was targeted, and all cryptographic recovery and decryption activities were performed offline against the supplied evidence.**
