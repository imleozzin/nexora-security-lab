# Passo a passo — Fase 5: Resposta a Incidentes

> Status: ⬜ A executar. Objetivo: transformar cada detecção da Fase 4 num relatório de incidente no padrão de SOC. Esta é a fase que mais impressiona recrutador.

> Pré-requisito: Fase 4 (detecções feitas e com prints).

---

## O ciclo de resposta (framework NIST, simplificado)
1. **Detecção** — o alerta disparou (Wazuh).
2. **Triagem** — é real ou falso positivo? Qual severidade?
3. **Contenção** — parar o dano (bloquear conta, isolar host, cortar regra).
4. **Erradicação** — remover a causa (conta backdoor, artefato).
5. **Recuperação** — voltar ao normal com segurança.
6. **Lições aprendidas** — o que muda pra não repetir.

---

## Para cada incidente, faça:

### Passo 1 — Abrir o relatório
Copie `06-relatorios-incidente/template-relatorio.md` para `INC-00X-nome.md`.

### Passo 2 — Preencher com as evidências reais
Puxe do Wazuh e dos Event Viewer:
- Event IDs, horários exatos → monte a **timeline**
- Prints do alerta e dos logs → referencie em `evidencias/`
- IP de origem, conta alvo, host afetado

### Passo 3 — Triagem e severidade
Classifique: Baixa / Média / Alta / Crítica. Justifique (ex.: "Alta — envolveu grupo Domain Admins").

### Passo 4 — Contenção (execute e documente)
Exemplos por cenário:
- Brute force → `Disable-ADAccount` na conta alvo; bloquear IP de origem no pfSense.
- Admin backdoor → `Remove-ADGroupMember "Domain Admins" backdoor`; `Disable-ADAccount backdoor`.
- PowerShell suspeito → isolar o host (tirar da rede), coletar artefatos.

### Passo 5 — Recomendações
O que evitaria/reduziria: MFA, baixar limite de lockout, alerta de correlação, tiering de contas admin.

### Passo 6 — Lições aprendidas
Técnico E de processo. Ex.: "a auditoria precisava estar ligada ANTES — ordenar hardening antes de monitorar."

---

## Mapeie tudo ao MITRE ATT&CK
Cada incidente cita a técnica (T1110, T1136, T1059.001...). Um quadro-resumo no fim liga seus incidentes às táticas (Initial Access, Persistence, Privilege Escalation, Discovery). Isso mostra maturidade.

---

## Entregável final da fase
- 1 relatório por cenário da Fase 4 (INC-001 a INC-006)
- Um `README.md` na pasta ligando os incidentes ao MITRE

## Provas
- [ ] Relatórios preenchidos com evidências reais (não template vazio)
- [ ] Ações de contenção documentadas e testadas
- [ ] Mapa MITRE dos incidentes

## Commit
```bash
git add .
git commit -m "feat: relatorios de incidente INC-001..006 com contencao e MITRE"
git push
```

---

## 🎓 Ao terminar a Fase 5
O projeto está **completo e apresentável**. Aí sim:
- Preencher as "Decisões de Arquitetura" (os porquês)
- Revisar o README principal como vitrine
- Preparar o **post do LinkedIn** (a gente escreve juntos)
- Entrevista simulada: "conte um incidente que você investigou" → você terá 6 histórias reais.
