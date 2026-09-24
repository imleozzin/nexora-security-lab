# 03 — Hardening (GPOs e endurecimento)

> Status: ⬜ Não iniciada. Esqueleto a preencher na Fase 2.

Objetivo: aplicar menor privilégio e ligar a auditoria **antes** de monitorar — sem os logs certos, o SIEM não enxerga os ataques.

## GPOs planejadas

| # | GPO | Vinculada em (OU) | Objetivo | Status |
|---|---|---|---|---|
| 1 | Política de senha | domínio / Default Domain Policy | mín. 12 caracteres, bloqueio após 5 tentativas | ⬜ |
| 2 | Auditoria avançada | domínio | logon, gestão de contas, criação de processos (essencial p/ SIEM) | ⬜ |
| 3 | Desabilitar SMBv1 + LLMNR | Computadores | reduzir superfície e ataques de relay | ⬜ |
| 4 | PowerShell Script Block Logging | Computadores | registrar scripts (Event ID 4104) | ⬜ |
| 5 | Bloqueio de USB | OU Financeiro | prevenir exfiltração/execução via mídia removível | ⬜ |
| 6 | Banner de aviso legal | domínio | aviso no logon (mostra controle por GPO) | ⬜ |

## Para cada GPO, documentar

- Nome e onde foi vinculada (e **por quê** ali)
- Configurações exatas alteradas
- Como foi **comprovada** a aplicação (`gpresult /r`, `gpresult /h relatorio.html`, `rsop.msc`)
- Print da política aplicada na estação

## Permissões de pasta (RBAC)

- [ ] Cada departamento acessa apenas a própria pasta em `\\SRV-FILE01`
- [ ] Grupos `GRP-*` como base das permissões (nunca usuários individuais)

---

## 🎯 Desafio pendente (mentor)

Responder no caderno / commit:
1. Qual GPO você já criou (real ou lab) e em que contexto?
2. Em qual OU vinculou e por quê?
3. Como **provou** que aplicou? Qual comando?
