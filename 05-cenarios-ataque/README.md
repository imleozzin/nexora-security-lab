# 05 — Cenários de ataque e detecção

> Status: ⬜ Não iniciada. Esqueleto a preencher na Fase 4.
>
> ⚠️ Todos os ataques partem do **KALI01** na zona isolada, contra alvos do próprio lab, com a regra liberada no pfSense apenas durante o teste.

## Cenários planejados

| # | Cenário | Ferramenta | Deve gerar | Status |
|---|---|---|---|---|
| 1 | Brute force em uma conta | Hydra / NetExec | vários 4625 → bloqueio | ⬜ |
| 2 | Password spraying | NetExec | 4625 em muitas contas | ⬜ |
| 3 | Criação de admin suspeito | PowerShell no DC | 4720 + 4728 | ⬜ |
| 4 | PowerShell ofuscado | WS-RH01 | 4104 + Sysmon 1 | ⬜ |
| 5 | Scan de rede | Nmap | logs do pfSense | ⬜ |
| 6 | Enumeração do AD | BloodHound | consultas LDAP | ⬜ |

## Fluxo de cada cenário

1. **Preparar:** snapshot `pre-ataque-X`, liberar regra no pfSense.
2. **Executar:** rodar o ataque a partir do Kali (comando exato documentado).
3. **Detectar:** localizar o alerta correspondente no Wazuh e tirar print.
4. **Mapear:** relacionar a técnica ao **MITRE ATT&CK** (ex.: T1110 Brute Force).
5. **Responder:** gerar o relatório em `06-relatorios-incidente/`.
6. **Restaurar:** voltar o snapshot e re-bloquear a regra.

## Template por cenário (copiar para um arquivo `cenario-0X-nome.md`)

```markdown
# Cenário X — <nome>

- **Objetivo:**
- **MITRE ATT&CK:** Txxxx
- **Ferramenta / comando:**
- **Alvo:**
- **Regra pfSense liberada:** (qual, por quanto tempo)

## Execução
(passos + saída da ferramenta)

## Detecção no Wazuh
(regra/alerta acionado + Event IDs + print)

## Análise
(o que os logs mostram, timeline)
```
