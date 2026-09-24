# Passo a passo — Zona de Usuários (firewall + DHCP + relay + ingresso)

> Status: 🟡 **Em andamento.** Firewall, DHCP e relay configurados; falta ingressar o WS-RH01.
>
> Objetivo: fazer a zona de Usuários funcionar — o WS-RH01 pega IP do próprio DC e entra no domínio `nexora.local`.

---

## Mapa de nomes (importante)

O pfSense mostra as interfaces pelos nomes de fábrica. Mapeamento lógico:

| Nome lógico | Aba no pfSense | Sub-rede |
|---|---|---|
| SERVIDORES | LAN | 192.168.10.0/24 |
| USUARIOS | OPT1 | 192.168.20.0/24 |
| SEGURANCA | OPT2 | 192.168.30.0/24 |
| ATAQUE | OPT3 | 192.168.99.0/24 |

> Dica: renomeie em **Interfaces → Assignments** (campo Description) para as abas mostrarem USUARIOS/SEGURANCA/ATAQUE. Fica mais legível e evita erro.

---

## Parte 1 — Regras de firewall (interface USUARIOS / OPT1)

Interface nova bloqueia tudo por padrão. O pfSense lê as regras **de cima para baixo; a primeira que casar vence.** Crie nesta ordem:

| # | Ação | Protocolo | Origem | Destino | Descrição |
|---|---|---|---|---|---|
| 1 | Pass | **Any** | USUARIOS net | **Single host** `192.168.10.10` | Serviços de domínio (TEMPORÁRIO) |
| 2 | Block | Any | USUARIOS net | `192.168.30.0/24` | Proteger zona de Segurança/SIEM |
| 3 | Block | Any | USUARIOS net | `192.168.99.0/24` | Isolar zona de Ataque |
| 4 | Block | Any | USUARIOS net | `192.168.10.0/24` | Bloquear resto de Servidores |
| 5 | Pass | Any | USUARIOS net | any | Navegação (internet) |

### Dois erros clássicos (aprendidos na prática)
- **Protocolo TCP não basta.** DNS (UDP 53), Kerberos (UDP/TCP 88) e DHCP (UDP) usam UDP. Use **Any** aqui; refina protocolo na Fase 2.
- **Máscara importa.** A regra 1 tem que ser **Single host `/32`** (só o DC). Se usar `/24`, ela libera a rede inteira e anula a regra 4.

> ⚠️ Dívida técnica proposital: a regra 1 libera o DC inteiro (ingresso no domínio usa RPC com portas dinâmicas). Apertar na Fase 2 (hardening).

---

## Parte 2 — DHCP Relay (pfSense)

**Services → DHCP Relay:**
- Enable: ✅
- Interface: **USUARIOS**
- Destination server: `192.168.10.10`
- Save.

**Por quê relay?** Na empresa quem entrega IP é o SRV-DC01 (DHCP integrado ao AD). O broadcast do cliente não cruza redes sozinho; o pfSense escuta e reencaminha ao DC. Termo: *DHCP relay agent*.

---

## Parte 3 — DHCP no SRV-DC01 (PowerShell admin)

```powershell
Install-WindowsFeature DHCP -IncludeManagementTools
Add-DhcpServerInDC -DnsName "srv-dc01.nexora.local" -IPAddress 192.168.10.10
Add-DhcpServerv4Scope -Name "USUARIOS" -StartRange 192.168.20.100 -EndRange 192.168.20.200 -SubnetMask 255.255.255.0
Set-DhcpServerv4OptionValue -ScopeId 192.168.20.0 -Router 192.168.20.1 -DnsServer 192.168.10.10 -DnsDomain "nexora.local"
```

> 🎯 A opção **Router tem que ser `192.168.20.1`** (gateway da rede do cliente), NÃO `.10.1`. E **DNS = 192.168.10.10** (o DC), nunca o pfSense.
>
> ℹ️ Se `Add-DhcpServerv4Scope` retornar `ResourceExists` (erro 20052), o escopo já existe — não é falha. Verifique com `Get-DhcpServerv4Scope`.

**Verificação:**
```powershell
Get-DhcpServerv4Scope                          # State deve ser Active
Get-DhcpServerv4OptionValue -ScopeId 192.168.20.0
```
Se State = Inactive: `Set-DhcpServerv4Scope -ScopeId 192.168.20.0 -State Active`

---

## Parte 4 — WS-RH01 no domínio

1. Ligar o WS-RH01 (placa em **VMnet11**).
2. `ipconfig /all` **no cliente** → esperar IP `192.168.20.100–200`, **DHCP Server 192.168.10.10**, DNS `192.168.10.10`.
3. `nslookup nexora.local` → deve responder `192.168.10.10`.
4. Ingressar:
```powershell
Add-Computer -DomainName "nexora.local" -Credential (Get-Credential) -Restart
```
Credencial: `NEXORA\Administrator`.
5. Após reiniciar, logar com conta do domínio → `whoami` mostra `nexora\...`.

📸 Snapshots: `WS-RH01-01-no-dominio`, `DC01-02-dhcp-ok`.

---

## Provas
- [ ] `ipconfig /all` do WS-RH01 com DHCP Server = 192.168.10.10
- [ ] `whoami` no cliente mostrando `nexora\...`

## Este passo valida três coisas de uma vez
Se o WS-RH01 pega IP do DC e entra no domínio, prova que: a regra de firewall Usuários→DC está certa, o DHCP Relay funciona, e o escopo+DNS estão corretos.

---

## Troubleshooting
| Sintoma | Causa provável | Ação |
|---|---|---|
| Cliente pega `169.254.x.x` (APIPA) | relay/escopo | conferir Destination do relay e `Get-DhcpServerv4Scope` |
| IP ok, sem internet | opção Router errada (.10.1) | corrigir para `192.168.20.1` |
| IP ok, não entra no domínio | firewall regra 1 ou DNS do cliente | DNS do cliente = 192.168.10.10; regra Pass ao DC ativa |
| "domain not found" | DNS errado | `nslookup nexora.local` deve responder .10.10 |
