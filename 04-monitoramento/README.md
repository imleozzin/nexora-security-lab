# 04 — Monitoramento (Wazuh + Sysmon)

> Status: ⬜ Não iniciada. Esqueleto a preencher na Fase 3.

Objetivo: centralizar logs no SIEM e garantir telemetria suficiente para detectar os ataques da Fase 4.

## Componentes

| Componente | Onde | Função |
|---|---|---|
| Wazuh (manager + dashboard) | SRV-SIEM (192.168.30.50) | SIEM: coleta, correlaciona, alerta |
| Agente Wazuh | DC01, FILE01, WS-RH01, WS-FIN01 | envia logs ao manager (1514/1515) |
| Sysmon | todos os Windows | telemetria de processos, rede, arquivos |
| Syslog | pfSense → SIEM | logs de firewall |

## Passos (a detalhar)

- [ ] Importar a OVA do Wazuh, colocar na VMnet12, IP `192.168.30.50`
- [ ] Acessar o dashboard e trocar senha padrão
- [ ] Instalar agente no DC01 e confirmar "Active" no painel
- [ ] Instalar agente nas demais máquinas
- [ ] Instalar Sysmon com config **SwiftOnSecurity** (`sysmon -accepteula -i sysmonconfig.xml`)
- [ ] Configurar Wazuh para coletar o canal Sysmon
- [ ] Encaminhar logs do pfSense via syslog remoto
- [ ] Validar que eventos chegam (gerar um logon de teste e achar no painel)

## Event IDs de referência

| Event ID | Significado |
|---|---|
| 4624 / 4625 | logon com sucesso / falha |
| 4720 | usuário criado |
| 4728 / 4732 | usuário adicionado a grupo privilegiado |
| 4104 | PowerShell Script Block |
| Sysmon 1 | criação de processo |
| Sysmon 3 | conexão de rede |
