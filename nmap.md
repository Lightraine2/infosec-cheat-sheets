# Nmap Security Testing Reference Guide

## Basic Scanning Examples



```bash

#IppSec's one nmap win:

nmap -sC -sV -p- -vv -oA scan 10.0.0.1

# Network ping sweep
nmap -sP 10.0.0.0/24

# Full TCP port scan with service detection
nmap -p 1-65535 -sV -sS -T4 target

# Aggressive scan with OS detection
nmap -v -sS -A -T4 target

# Full port range scan with OS detection
nmap -v -p 1-65535 -sV -O -sS -T4 target
```

> **Note**: Aggressive scan timings (T5) are faster but may yield inaccurate results. T3-T4 provides better compromise between speed and accuracy.

## Input/Output Options

### Scan from File
```bash
# Scan from IP list
nmap -iL ip-addresses.txt
```

### Output Formats
```bash
# Output all formats
nmap -sS -sV -T5 target -oA output-all-formats

# Grepable output
nmap -sV -p 139,445 -oG grep-output.txt 10.0.1.0/24

# HTML output
nmap -sS -sV -T5 target --webxml -oX - | xsltproc --output file.html -
```

## Specialized Scans

### Netbios Scanning
```bash
# Find Netbios servers
nmap -sV -v -p 139,445 10.0.1.0/24

# Display Netbios name
nmap -sU --script nbstat.nse -p 137 target

# Check for MS08-067
nmap --script-args=unsafe=1 --script smb-check-vulns.nse -p 445 target
```

### Web Server Scanning
```bash
# Scan HTTP servers and pipe to Nikto
nmap -p80 10.0.1.0/24 -oG - | nikto.pl -h -

# Scan HTTP/HTTPS
nmap -p80,443 10.0.1.0/24 -oG - | nikto.pl -h -
```

## Target Specification Options

| Option | Description |
|--------|-------------|
| `-iL` | Input from list of hosts/networks |
| `-iR` | Choose random targets |
| `--exclude` | Exclude hosts/networks |
| `--excludefile` | Exclude hosts from file |

## Host Discovery Options

| Option | Description |
|--------|-------------|
| `-sL` | List scan - simply list targets |
| `-sn` | Ping scan - disable port scan |
| `-Pn` | Treat all hosts as online |
| `-PS/PA/PU/PY` | TCP SYN/ACK, UDP or SCTP discovery |
| `-PE/PP/PM` | ICMP echo, timestamp, netmask request |
| `-n/-R` | Never do DNS resolution/Always resolve |

## Scan Techniques

| Option | Description |
|--------|-------------|
| `-sS` | TCP SYN scan |
| `-sT` | Connect scan |
| `-sU` | UDP Scan |
| `-sA` | ACK scan |
| `-sW` | Window scan |
| `-sM` | Maimon scan |
| `-sN` | TCP Null scan |
| `-sF` | FIN scan |
| `-sX` | Xmas scan |
| `-sO` | IP protocol scan |

## Port Specification

| Option | Description |
|--------|-------------|
| `-p` | Specify ports (e.g., -p80,443) |
| `-p U:PORT` | Scan UDP ports |
| `-F` | Fast mode - fewer ports |
| `-r` | Scan ports consecutively |
| `--top-ports` | Scan most common ports |

## Service/Version Detection

| Option | Description |
|--------|-------------|
| `-sV` | Probe for service/version info |
| `--version-intensity` | Set from 0 (light) to 9 (all probes) |
| `--version-light` | Limit to most likely probes |
| `--version-all` | Try every single probe |

## Script Scanning

| Option | Description |
|--------|-------------|
| `-sC` | Equivalent to --script=default |
| `--script` | Specify Lua scripts to run |
| `--script-args` | Provide arguments to scripts |
| `--script-updatedb` | Update script database |

## Performance Tuning

### Timing Templates
```bash
-T0  # Paranoid
-T1  # Sneaky
-T2  # Polite
-T3  # Normal
-T4  # Aggressive
-T5  # Insane
```

### Fine-Tuning Options
```bash
--min-rate <number>    # Send packets no slower than <number> per second
--max-rate <number>    # Send packets no faster than <number> per second
--min-parallelism <number>  # Minimum probe parallelization
--max-parallelism <number>  # Maximum probe parallelization
--min-hostgroup <number>    # Minimum parallel hosts
--max-hostgroup <number>    # Maximum parallel hosts
```

## IDS Evasion and Spoofing

| Option | Description |
|--------|-------------|
| `-f` | Fragment packets |
| `-D` | Cloak scan with decoys |
| `-S` | Spoof source address |
| `--data-length` | Append random data to packets |
| `--ttl` | Set IP time-to-live field |
| `--spoof-mac` | Spoof MAC address |
| `--badsum` | Send packets with bogus checksum |

## Best Practices

1. **Scan Accuracy**
   - Use T3/T4 timing for better accuracy
   - Avoid T5 timing unless speed is critical
   - Consider network conditions when selecting timing

2. **Performance Optimization**
   - Use `-n` to disable DNS resolution when not needed
   - Adjust parallelism based on network capacity
   - Use appropriate timing templates for target network

3. **IDS Evasion**
   - Use `--scan-delay` to avoid detection
   - Consider fragmenting packets with `-f`
   - Use decoy scans when appropriate

4. **Output Management**
   - Use `-oA` for comprehensive output
   - Consider grepable output for parsing
   - Use XML output for automated processing

## Troubleshooting

1. **Scan Taking Too Long**
   - Check for blacklisting
   - Verify network connectivity
   - Adjust timing templates
   - Consider reducing scope

2. **Missing Ports**
   - Verify timing isn't too aggressive
   - Check for filtering/firewalls
   - Try different scan techniques
   - Adjust probe timing

3. **False Positives/Negatives**
   - Reduce scan speed
   - Use multiple scan techniques
   - Verify results manually
   - Consider network conditions