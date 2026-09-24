# Troubleshooting & Lições Aprendidas

> O registro mais valioso do repositório para entrevistas. Formato: **Sintoma → Investigação → Causa raiz → Correção → Lição.**

---

## TS-01 — SRV-DC01 não alcançava o gateway ("Destination host unreachable")

- **Sintoma:** ping de `SRV-DC01` para `192.168.10.1` retornava *Destination host unreachable*; sem rede.
- **Investigação:** erro de ARP → problema de **camada 2**. Comparei os MACs das placas do firewall com as interfaces do pfSense.
- **Causa raiz:** no FW01, as **Network Adapters 2 e 4 estavam em VMnets trocadas** (a LAN `192.168.10.1` caiu no segmento de Segurança).
- **Correção:** ajustei as VMnets no VMware (Adapter 2 → VMnet10, Adapter 4 → VMnet12).
- **Lição:** validar o mapeamento **MAC ↔ interface ↔ VMnet** antes de configurar serviços.
- **Vocabulário:** *layer 2 issue, ARP resolution failed, no carrier.*

---

## TS-02 — Gateway alcançável, mas sem internet no SRV-DC01

- **Sintoma:** `Test-NetConnection 192.168.10.1` = True, mas o navegador não abria sites.
- **Investigação:** testei em camadas — ping por IP (ok) vs. resolução de nomes (falha). Isolei que era **DNS**.
- **Causa raiz:** DNS do DC apontava para `127.0.0.1` **antes** de o serviço DNS existir (pré-promoção).
- **Correção:** DNS temporário para o pfSense (`192.168.10.1`); na promoção o Windows volta para loopback.
- **Lição:** "sem internet" nem sempre é falta de conectividade — muitas vezes é **DNS**.
- **Vocabulário:** *name resolution, DNS forwarder, loopback.*

---

## TS-03 — dcdiag falhou DFSREvent e SystemLog após a promoção

- **Sintoma:** `dcdiag /q` retornou `failed test DFSREvent` e `failed test SystemLog`.
- **Investigação:** triagem dos eventos. SystemLog sinaliza **qualquer** erro do sistema nas últimas 24h.
- **Causa raiz:** ruído pós-promoção — shutdown não-limpo (snapshot), falhas de update da Microsoft Store, e um evento KDC transitório. DFSREvent = inicialização normal do SYSVOL dentro da janela de 24h.
- **Correção:** reboot (limpou o SystemLog). SYSVOL saudável confirmado por outra via: `net share` (SYSVOL e NETLOGON presentes) e log DFS Replication sem eventos de erro (só Information).
- **Lição:** *"failed" nem sempre é falha.* dcdiag SystemLog exige interpretação, não pânico. Confirmar a saúde pela fonte, não só pelo teste.
- **Vocabulário:** *false positive, SYSVOL replication, signal vs. noise.*

---

## TS-04 — SRV-DC01 sem internet (WAN sem gateway)

- **Sintoma:** DC perdeu acesso à internet. `Resolve-DnsName google.com` = DNS server failure.
- **Investigação:** camada por camada — DC→pfSense (`ping .10.1`) OK, mas `ping 8.8.8.8` = *DestinationHostUnreachable* (o próprio pfSense respondendo "sem rota"). Fui ao pfSense: Status → Gateways sem gateway WAN.
- **Causa raiz:** a interface **WAN estava como Static IPv4** (`192.168.192.128/24`) com **Upstream gateway = None**. IP sem gateway = **sem rota default**. Antes funcionava porque a WAN estava em DHCP e recebia IP + gateway do NAT do VMware.
- **Correção:** Interfaces → WAN → IPv4 Configuration Type = **DHCP** → Save → Apply. O NAT do VMware voltou a entregar IP e gateway.
- **Lição:** **um IP na interface não é conectividade.** Sem gateway (próximo salto) não há rota. Endereço sem rota é casa sem porta de saída.
- **Vocabulário:** *default route, upstream gateway, next hop.*

---

## TS-0X — (modelo para os próximos)

- **Sintoma:**
- **Investigação:**
- **Causa raiz:**
- **Correção:**
- **Lição:**
