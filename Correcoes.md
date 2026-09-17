Aqui estão as recomendações de correções para as falhas encontradas durante o pentest, organizadas por tipo de rede:

---

## 🔒 WEP
- **Substituir WEP por protocolos modernos:** migrar para WPA2 ou WPA3, já que WEP é obsoleto e facilmente quebrado.  
- **Desativar redes legadas:** remover SSIDs antigos que ainda utilizam WEP.  
- **Segmentação de rede:** caso seja necessário manter dispositivos antigos, isolá-los em VLANs restritas sem acesso crítico.  

---

## 🔑 WPA/WPA2‑PSK
- **Política de senhas fortes:** substituir senhas fracas ou previsíveis (ex.: “starwars1”) por senhas longas, complexas e únicas.  
- **Rotação periódica de senhas:** implementar troca regular de chaves PSK.  
- **Autenticação centralizada:** migrar para WPA2‑Enterprise ou WPA3‑Enterprise, evitando senhas compartilhadas entre usuários.  
- **Monitoramento de deauth:** configurar sistemas de detecção de intrusão wireless (WIDS) para identificar ataques de desautenticação.  

---

## 🛡️ WPA‑Enterprise (802.1X/EAP)
- **Validação de certificados:** garantir que clientes validem corretamente certificados do servidor RADIUS, evitando ataques Evil Twin.  
- **Certificados confiáveis:** usar certificados emitidos por autoridades reconhecidas e não autoassinados.  
- **Proteção contra NTLM relay:** aplicar políticas de autenticação modernas (Kerberos, certificados) e desabilitar NTLM quando possível.  
- **Treinamento de usuários:** conscientizar sobre riscos de redes falsas e boas práticas de conexão.  
- **Monitoramento de acessos:** implementar logs e alertas para tentativas suspeitas de autenticação.  

---

## 📌 Recomendações Gerais
- **Migrar para WPA3:** sempre que possível, adotar WPA3, que oferece proteção contra ataques de dicionário offline e forward secrecy.  
- **Segmentação de tráfego:** separar redes corporativas, convidado e IoT em diferentes VLANs.  
- **WIDS/WIPS:** implementar sistemas de detecção e prevenção de intrusão wireless.  
- **Políticas de segurança:** documentar e aplicar políticas claras de uso de Wi‑Fi, incluindo requisitos de senha e autenticação.  
- **Atualizações de firmware:** manter roteadores e pontos de acesso sempre atualizados para corrigir vulnerabilidades conhecidas.  

---

👉 Essas recomendações devem ser apresentadas como **plano de mitigação** no relatório final, mostrando não apenas a exploração das falhas, mas também a proposta de correção e endurecimento da segurança. 
