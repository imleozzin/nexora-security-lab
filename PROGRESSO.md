# 📊 Progresso do projeto

> Quadro vivo. Atualize a cada sessão: marque `[x]`, mude o status e anote a data.

## Quadro de fases

| Fase | Descrição | Status | Data |
|---|---|---|---|
| 1 | Construir (rede + AD) | 🟡 Em andamento (~60%) | — |
| 2 | Proteger (hardening + GPOs) | ⬜ | — |
| 3 | Monitorar (Wazuh + Sysmon) | ⬜ | — |
| 4 | Atacar e detectar | ⬜ | — |
| 5 | Responder (relatórios) | ⬜ | — |

---

## ✅ Checklist detalhado

### Fase 1 — Construir
- [x] Criar VMnets 10/11/12/13 (host-only, sem DHCP)
- [x] Instalar pfSense (FW01) com 5 interfaces
- [x] Configurar IPs das interfaces
- [x] Instalar Windows Server (SRV-DC01)
- [x] Rede do DC OK (após corrigir VMnets — TS-01 — e DNS — TS-02)
- [x] **Promover a Domain Controller (`nexora.local`) — verificado** (TS-03)
- [x] DNS Forwarder para o pfSense
- [x] Renomear interfaces do pfSense (USUARIOS/SEGURANCA/ATAQUE)
- [x] Regras de firewall na zona USUARIOS (Any + Single host DC + blocks)
- [x] DHCP Relay no pfSense (USUARIOS → 192.168.10.10)
- [x] DHCP no SRV-DC01 (escopo USUARIOS, opções 003/006/015, Active)
- [x] Corrigir WAN do pfSense (Static→DHCP — TS-04)
- [ ] WS-RH01 pega IP via DHCP e ingressa no domínio
- [ ] Estrutura de OUs (script criar-estrutura-ad.ps1)
- [ ] Usuários e grupos
- [ ] Ubuntu SRV-FILE01
- [ ] Confirmar/corrigir acesso HTTPS ao pfSense

### Fase 2 — Proteger
- [ ] GPO: política de senha
- [ ] GPO: auditoria avançada (essencial p/ o SIEM)
- [ ] GPO: desabilitar SMBv1 e LLMNR
- [ ] GPO: PowerShell Script Block Logging
- [ ] GPO: bloqueio de USB (Financeiro)
- [ ] GPO: banner de aviso legal
- [ ] Apertar regras de firewall (fim do "allow any" temporário)
- [ ] Permissões de pasta por departamento (RBAC)

### Fase 3 — Monitorar
- [ ] Subir SRV-SIEM (Wazuh)
- [ ] Agente Wazuh em DC01, FILE01, WS-RH01, WS-FIN01
- [ ] Sysmon (config SwiftOnSecurity) nos Windows
- [ ] Logs do pfSense via syslog
- [ ] Confirmar logs no painel

### Fase 4 — Atacar e detectar
- [ ] Brute force · Password spraying · Admin suspeito · PowerShell ofuscado · Nmap · BloodHound

### Fase 5 — Responder
- [ ] 1 relatório de incidente por cenário

---

## 🔬 Provas da Fase 1 (salvar em evidencias/)

| # | Prova | Comando | Status |
|---|---|---|---|
| 1 | Domínio criado | `Get-ADDomain` | ✅ |
| 2 | DC saudável | `dcdiag` + `net share` (SYSVOL/NETLOGON) | ✅ |
| 3 | DNS resolve domínio | `Resolve-DnsName srv-dc01.nexora.local` | ✅ |
| 4 | Internet via pfSense | `Resolve-DnsName google.com` | ✅ (após TS-04) |
| 5 | DHCP configurado | `Get-DhcpServerv4Scope` (Active) | ✅ |
| 6 | Cliente pega IP do DC | `ipconfig /all` no WS-RH01 | ⬜ |
| 7 | Cliente no domínio | `whoami` = nexora\... | ⬜ |
| 8 | Kali NÃO alcança nada | `ping 192.168.10.10` do Kali (deve falhar) | ⬜ |

---

## 📸 Snapshots

| VM | Snapshot | Quando |
|---|---|---|
| FW01 | `FW01-01-interfaces-configuradas` | ✅ |
| SRV-DC01 | `DC01-00b-rede-ok` | ✅ |
| SRV-DC01 | `DC01-01-dc-promovido` | ✅ |
| SRV-DC01 | `DC01-02-dhcp-ok` | pendente |
| WS-RH01 | `WS-RH01-01-no-dominio` | pendente |

---

## 📈 Quadro de evolução

| Área | Nível atual | Meta |
|---|---|---|
| Redes | 2→3 | 3 – Júnior empregável |
| Active Directory | 2 | 3 |
| Troubleshooting | 2→3 | 3 |
| Linux | 1 | 3 |
| SOC / SIEM | 0 | 3 |
| Documentação | 2 | 3 |
| GitHub | 1 | 3 |

---

## 📌 Pendências (mentor cobra)

- [ ] Escrever "Decisões de Arquitetura" (ADR-02..05) com as próprias palavras
- [ ] Responder: cenário A ou B no TS-02? Por quê?
- [ ] Desafio da GPO: qual criou, onde vinculou, como provou
- [ ] Publicar no GitHub (git push do PC) + renomear repo para `nexora-security-lab`
