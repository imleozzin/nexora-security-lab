# Passo a passo — Fase 4: Atacar e detectar

> Status: ⬜ A executar. Objetivo: gerar ataques controlados a partir do KALI01 e **encontrar cada um no Wazuh**. O foco é a DETECÇÃO (Blue Team), não o ataque em si.

> ⚠️ **ÉTICA E ESCOPO — leia:** tudo aqui roda **exclusivamente** contra as VMs do seu próprio laboratório, em rede isolada, sem internet na zona de Ataque. Atacar sistemas de terceiros é crime. Este exercício existe para você aprender a **detectar e responder** — que é o trabalho de um SOC.

> Pré-requisito: Fase 3 no ar (Wazuh + Sysmon coletando).

---

## Preparação (a cada cenário)
1. Snapshot `pre-ataque-X` nas VMs alvo.
2. No pfSense, libere **temporariamente** a regra ATAQUE → alvo (só durante o teste).
3. Execute o ataque.
4. **Ache o alerta no Wazuh** e tire print.
5. Mapeie a técnica no **MITRE ATT&CK**.
6. Re-bloqueie a regra e volte o snapshot.
7. Escreva o relatório em `06-relatorios-incidente/`.

---

## Cenário 1 — Brute force (T1110.001)
A partir do KALI01, contra a conta de um usuário do domínio:
```bash
# hydra contra SMB (exemplo didático, contra o SEU lab)
hydra -l maria.silva -P /usr/share/wordlists/rockyou.txt smb://192.168.10.10
```
**Deve gerar:** muitos Event ID **4625** e, ao atingir o limite, **4740** (lockout).
**No Wazuh:** regra de "multiple authentication failures". Print.

## Cenário 2 — Password spraying (T1110.003)
Uma senha comum contra várias contas (evita lockout):
```bash
# NetExec (sucessor do CrackMapExec)
nxc smb 192.168.10.10 -u usuarios.txt -p 'Verao@2026'
```
**Deve gerar:** 4625 espalhado por várias contas, mesma origem.
**Detecção:** correlação "muitas contas, um IP" — mais sutil que brute force.

## Cenário 3 — Criação de admin suspeito (T1136 / T1098)
No DC (simulando pós-comprometimento), via PowerShell:
```powershell
New-ADUser -Name "backdoor" -SamAccountName backdoor -AccountPassword (ConvertTo-SecureString "P@ss!2026" -AsPlainText -Force) -Enabled $true
Add-ADGroupMember -Identity "Domain Admins" -Members backdoor
```
**Deve gerar:** **4720** (user criado) + **4728** (add a Domain Admins).
**Detecção:** alerta de mudança em grupo privilegiado — dos mais importantes num SOC real.

## Cenário 4 — PowerShell ofuscado (T1059.001 / T1027)
No WS-RH01:
```powershell
powershell -enc <comando_base64_inofensivo>   # ex.: um Write-Host codificado
```
**Deve gerar:** **4104** (script block) + Sysmon **1** (processo com linha de comando suspeita).
**Detecção:** encoding/`-enc` é red flag clássico.

## Cenário 5 — Scan de rede (T1046)
```bash
nmap -sS -p- 192.168.10.0/24
```
**Deve gerar:** logs de firewall no pfSense (muitas conexões, muitas portas).
**Detecção:** varredura horizontal/vertical.

## Cenário 6 — Enumeração do AD (T1087 / T1069)
```bash
# BloodHound coletor (no lab)
bloodhound-python -d nexora.local -u maria.silva -p 'senha' -c all -ns 192.168.10.10
```
**Deve gerar:** volume anômalo de consultas LDAP.
**Detecção:** reconhecimento de AD — antecede movimentação lateral.

---

## Para CADA cenário, documente (copie de 05-cenarios-ataque/README.md)
- Objetivo, MITRE ID, comando exato
- Saída da ferramenta (print)
- Alerta/Event IDs no Wazuh (print)
- Timeline e análise

## Provas
- [ ] 1 print de detecção no Wazuh por cenário
- [ ] Mapeamento MITRE de cada um

## Commit
```bash
git add .
git commit -m "feat: cenarios de ataque 1-6 com deteccao no Wazuh e mapeamento MITRE"
git push
```

Próximo: **Fase 5 — Resposta a incidentes** (transformar cada detecção em relatório).
