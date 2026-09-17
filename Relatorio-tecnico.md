# Relatório Técnico – Exame OSWP

## Introdução
O presente relatório documenta as atividades realizadas durante a avaliação prática do **OffSec Wireless Professional (OSWP)**. O objetivo é demonstrar conhecimento técnico e metodológico na exploração de diferentes tipos de redes sem fio, abrangendo **WEP**, **WPA/WPA2‑PSK** e **WPA‑Enterprise (802.1X/EAP)**.  

Cada seção descreve a metodologia aplicada, os comandos utilizados, os resultados obtidos e a comprovação de acesso à rede alvo.

---

## 1. Metodologia Geral
A abordagem seguiu as seguintes etapas:
- **Reconhecimento:** Identificação de redes disponíveis, coleta de informações de SSID, BSSID, canal e tipo de criptografia.  
- **Captura:** Utilização do modo monitor para interceptar pacotes relevantes.  
- **Ataque:** Aplicação de técnicas específicas para cada protocolo.  
- **Quebra:** Processamento dos pacotes capturados para extração de chaves ou credenciais.  
- **Conexão:** Configuração de cliente para autenticação na rede alvo.  
- **Prova:** Demonstração de acesso à rede, evidenciando a eficácia da exploração.

---

## 2. Redes WEP
### Procedimentos
1. Preparação da interface em modo monitor.  
2. Captura de pacotes da rede alvo.  
3. Execução de ataque ARP replay para geração de tráfego.  
4. Quebra da chave WEP com `aircrack-ng`.  
5. Conexão à rede utilizando `wpa_supplicant` e obtenção de IP via DHCP.  
6. Prova de acesso mediante requisição HTTP ao roteador.

### Resultado
- Chave WEP obtida: **11BB33CD55**  
- Acesso confirmado através do arquivo `proof.txt`.

---

## 3. Redes WPA/WPA2‑PSK
### Procedimentos
1. Captura de handshake WPA/WPA2 com `airodump-ng`.  
2. Força de desconexão de clientes com `aireplay-ng`.  
3. Quebra da senha utilizando wordlist (`rockyou.txt`).  
4. Conexão à rede com `wpa_supplicant` e obtenção de IP via DHCP.

### Resultado
- Senha WPA2 obtida: **starwars1**  
- Conexão estabelecida com sucesso à rede alvo.

---

## 4. Redes WPA‑Enterprise (802.1X/EAP)
### Procedimentos
1. Reconhecimento e captura de tráfego EAP/PEAP.  
2. Extração de certificados com `tshark`.  
3. Configuração de ponto de acesso malicioso (Evil Twin) com **hostapd‑mana** e certificados gerados via FreeRADIUS.  
4. Captura de credenciais NTLM durante autenticação de clientes.  
5. Quebra de hashes com **John the Ripper** e **Hashcat**.  
6. Conexão à rede corporativa utilizando credenciais válidas.  

### Resultado
- Credenciais capturadas: `juan.tr`  
- Senha obtida: **bulldogs123**  
- Conexão estabelecida à rede corporativa `wifi-corp`.

---

## Conclusão
O presente relatório evidencia a aplicação prática de técnicas de exploração em três tipos distintos de redes sem fio:  
- **WEP:** vulnerável à coleta de IVs e ataques de reinjeção ARP.  
- **WPA/WPA2‑PSK:** suscetível à captura de handshakes e ataques de dicionário.  
- **WPA‑Enterprise:** exposto a ataques de engenharia (Evil Twin) e quebra de credenciais NTLM.  

A documentação comprova a capacidade técnica do candidato em identificar, explorar e validar vulnerabilidades em ambientes Wi‑Fi, atendendo aos requisitos do exame **OffSec Wireless Professional**.
---
