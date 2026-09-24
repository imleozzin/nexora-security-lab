# Passo a passo — SRV-DC01 (Windows Server + Active Directory)

> Status: 🟡 **Em andamento.** Rede OK; falta promover a DC.

---

## Etapa 1 — Baixar e criar a VM

- Baixar **Windows Server 2025 Evaluation (ISO)** no Microsoft Evaluation Center (180 dias, grátis).
- VM: **"I will install the OS later"** (evita o Easy Install). Firmware **UEFI**, 2 núcleos, 4 GB RAM, disco 60 GB, rede **Custom: VMnet10**. Apontar o CD/DVD para a ISO.

## Etapa 2 — Instalar o Windows Server

- Edição: **Standard Evaluation (Desktop Experience)** — ⚠️ não escolher Core (sem GUI).
- Custom install → disco inteiro → definir senha do Administrator (forte, anotada).
- Instalar **VMware Tools** e reiniciar.
- 📸 Snapshot: `DC01-00-instalacao-limpa`

## Etapa 3 — Rede (PowerShell como admin)

```powershell
# a) nome da placa
Get-NetAdapter

# b) IP fixo + DNS temporário (pfSense) — ver nota abaixo
New-NetIPAddress -InterfaceAlias "Ethernet0" -IPAddress 192.168.10.10 -PrefixLength 24 -DefaultGateway 192.168.10.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet0" -ServerAddresses 192.168.10.1

# c) fuso horário (Kerberos rejeita diferença > 5 min)
Set-TimeZone -Id "E. South America Standard Time"

# d) testar
Test-NetConnection 192.168.10.1        # PingSucceeded: True
Resolve-DnsName google.com             # deve resolver

# e) renomear
Rename-Computer -NewName "SRV-DC01" -Restart
```

> ⚠️ **Nota (aprendido na prática):** antes da promoção, o DNS aponta para o **pfSense** (`192.168.10.1`), NÃO para `127.0.0.1`. O loopback só passa a valer depois que o serviço DNS existir (na promoção). Apontar para loopback cedo demais = "sem internet". Ver `07-lessons-learned`.

📸 Snapshot: `DC01-00b-rede-ok`

## Etapa 4 — Promover a Domain Controller

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
Install-ADDSForest -DomainName "nexora.local" -DomainNetbiosName "NEXORA" -InstallDns
```
- Definir e **anotar** a senha do **DSRM** (recuperação do AD).
- Aviso de delegação de DNS: normal no lab, ignorar.
- Reinicia sozinho. Login passa a ser `NEXORA\Administrator`.

## Etapa 5 — DNS Forwarder (resolver a internet)

```powershell
Add-DnsServerForwarder -IPAddress 192.168.10.1
```

📸 Snapshot: `DC01-01-dc-promovido`

---

## Provas (salvar prints em `evidencias/`)

| # | Comando | Esperado |
|---|---|---|
| 1 | `Get-ADDomain \| Select DNSRoot, NetBIOSName` | `nexora.local` / `NEXORA` |
| 2 | `dcdiag /q` | sem erros |
| 3 | `Resolve-DnsName srv-dc01.nexora.local` | `192.168.10.10` |
| 4 | `Resolve-DnsName google.com` | IP público (forwarder OK) |

---

## Próximo

Regras de firewall por zona → DHCP + Relay → estrutura de OUs (`scripts/criar-estrutura-ad.ps1`).
