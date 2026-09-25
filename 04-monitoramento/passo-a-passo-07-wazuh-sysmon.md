# Passo a passo — Fase 3: Monitoramento (Wazuh + Sysmon)

> Status: ⬜ A executar. Objetivo: centralizar logs no SIEM (Wazuh) e ter telemetria (Sysmon) suficiente para detectar os ataques da Fase 4.

> Pré-requisito: Fase 2 concluída (auditoria e PowerShell logging ligados).

---

## Etapa 1 — Subir o SRV-SIEM (Wazuh)
Opção mais simples: **OVA all-in-one** da Wazuh (manager + indexer + dashboard).
- Importe a OVA no VMware → coloque a placa na **VMnet12 (Segurança)**.
- Recursos: 4 vCPU, 8 GB RAM (o Wazuh é pesado).
- No console, configure IP estático `192.168.30.50/24`, gateway `192.168.30.1`, DNS `192.168.10.10`.
- Acesse o dashboard: `https://192.168.30.50` (usuário/senha vêm na doc da OVA) → **troque a senha padrão**.

> Alternativa (mais leve p/ estudar): instalar via script em um Ubuntu:
> `curl -sO https://packages.wazuh.com/4.x/wazuh-install.sh && sudo bash ./wazuh-install.sh -a`

## Etapa 2 — Liberar o firewall p/ os agentes
No pfSense, garanta que as zonas conseguem falar com o SIEM nas portas **1514/1515 TCP** (agente → manager). Regra por zona → destino `192.168.30.50`, portas 1514-1515.

## Etapa 3 — Instalar o agente Wazuh nos Windows
No dashboard: **Agents → Deploy new agent** → escolha Windows → ele gera o comando. No DC01 e no WS-RH01 (PowerShell admin), algo como:
```powershell
# exemplo — use o comando exato que o dashboard gerar (com o IP do manager)
.\wazuh-agent.msi /q WAZUH_MANAGER="192.168.30.50" WAZUH_AGENT_NAME="SRV-DC01"
net start WazuhSvc
```
Repita para WS-RH01 e para o Ubuntu (SRV-FILE01, comando Linux do dashboard).

**Confirme:** no dashboard, os agentes aparecem como **Active**.

## Etapa 4 — Instalar o Sysmon (telemetria rica)
Nos Windows (DC01, WS-RH01):
```powershell
# baixe o Sysmon (Sysinternals) e a config da SwiftOnSecurity (sysmonconfig-export.xml)
.\Sysmon64.exe -accepteula -i sysmonconfig-export.xml
```
Config recomendada: **SwiftOnSecurity** ou **Olaf Hartong** (gratuitas no GitHub).

## Etapa 5 — Fazer o Wazuh coletar o canal Sysmon
No agente (arquivo `ossec.conf` ou via config central), adicione a coleta do canal:
```xml
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```
Reinicie o agente.

## Etapa 6 — Logs do pfSense (opcional mas recomendado)
pfSense → Status → System Logs → Settings → **Remote Logging** → servidor `192.168.30.50`. No Wazuh, configure a recepção de syslog.

---

## Validação (o teste que prova o pipeline)
1. No WS-RH01, gere um evento: erre a senha de logon 1x, ou rode `whoami` num PowerShell.
2. No dashboard do Wazuh, procure o evento (Security Events / Discover).
3. Se aparecer → o pipeline **coleta → indexa → mostra** está de pé.

📸 Print do dashboard com os agentes Active + um evento chegando → `04-monitoramento/evidencias/`.

---

## Event IDs de referência (memorize)
| ID | Significado |
|---|---|
| 4624 / 4625 | logon sucesso / falha |
| 4740 | conta bloqueada |
| 4720 | usuário criado |
| 4728 / 4732 | add a grupo privilegiado |
| 4688 | processo criado (com linha de comando) |
| 4104 | PowerShell Script Block |
| Sysmon 1 | criação de processo |
| Sysmon 3 | conexão de rede |

## Provas
- [ ] Agentes Active no dashboard (DC01, WS-RH01, FILE01)
- [ ] Sysmon instalado e coletado
- [ ] Evento de teste visível no Wazuh

## Commit
```bash
git add .
git commit -m "feat: monitoramento - Wazuh + agentes + Sysmon"
git push
```

Próximo: **Fase 4 — Atacar e detectar**.
