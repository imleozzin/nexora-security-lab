# Passo a passo — Fase 2: Hardening + GPOs

> Status: ⬜ A executar. Objetivo: aplicar menor privilégio e **ligar a auditoria** — sem isso, o SIEM da Fase 3 não enxerga ataque.

> ⚠️ Ordem importa: a GPO de auditoria (2) e a de PowerShell logging (4) são pré-requisito da Fase 3.

Todas as GPOs são criadas no SRV-DC01 → **Group Policy Management** (`gpmc.msc`).

---

## GPO 1 — Política de senha (domínio)
Editar a **Default Domain Policy** → Computer Config → Policies → Windows Settings → Security Settings → Account Policies → Password Policy:
- Minimum password length: **12**
- Password must meet complexity: **Enabled**
- Account Lockout Policy → Lockout threshold: **5** tentativas; duration/reset: 15 min.

**Prova:** `net accounts` no DC mostra os novos valores.

---

## GPO 2 — Auditoria avançada (a mais importante p/ o SIEM)
Nova GPO "AUDITORIA" vinculada ao domínio → Computer Config → Policies → Windows Settings → Security Settings → **Advanced Audit Policy Configuration**:
- Logon/Logoff → **Audit Logon** e **Audit Account Lockout**: Success+Failure
- Account Logon → **Audit Kerberos** e **Credential Validation**: Success+Failure
- Account Management → **Audit User Account Management** e **Security Group Management**: Success+Failure
- Detailed Tracking → **Audit Process Creation**: Success
- Também: Administrative Templates → System → Audit Process Creation → **Include command line**: Enabled (grava a linha de comando no 4688)

**Event IDs que isso liga:** 4624/4625 (logon), 4740 (lockout), 4720 (user criado), 4728/4732 (add a grupo), 4688 (processo criado).

---

## GPO 3 — Reduzir superfície (SMBv1 + LLMNR)
Nova GPO "HARDENING-REDE" vinculada a OU=Computadores:
- LLMNR off: Admin Templates → Network → DNS Client → **Turn off multicast name resolution**: Enabled
- SMBv1: desabilite via GPO de registro ou `Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol` nas máquinas.

**Por quê:** LLMNR/NBT-NS são vetor clássico de *poisoning* (Responder). SMBv1 é obsoleto e explorável (EternalBlue).

---

## GPO 4 — PowerShell Script Block Logging
Nova GPO "POWERSHELL-LOG" em OU=Computadores → Admin Templates → Windows Components → Windows PowerShell:
- **Turn on PowerShell Script Block Logging**: Enabled

**Event ID:** 4104. Essencial para detectar PowerShell ofuscado (Fase 4).

---

## GPO 5 — Bloqueio de USB (OU=Financeiro)
Nova GPO vinculada só à OU Financeiro → Computer Config → Admin Templates → System → Removable Storage Access → **All Removable Storage classes: Deny all access**: Enabled.

**Por quê:** demonstra política por OU (não vale pra empresa toda, só um setor) e previne exfiltração.

---

## GPO 6 — Banner de aviso legal
Default Domain Policy → Security Options:
- **Interactive logon: Message title** = "Aviso"
- **Message text** = "Acesso restrito a usuários autorizados. Uso monitorado."

Mostra domínio de GPO e é boa prática de compliance.

---

## Aplicar e provar
Nas estações:
```powershell
gpupdate /force
gpresult /h C:\gpresult.html    # abre e mostra as GPOs aplicadas
```
Confirme no WS-RH01 que a política de senha, o banner e o PowerShell logging vieram.

📸 Prints do `gpresult` e do banner no logon → `03-hardening/evidencias/`.

---

## Provas
- [ ] `net accounts` com senha mín. 12 / lockout 5
- [ ] `gpresult /h` listando as 6 GPOs
- [ ] Banner aparece no logon
- [ ] Event ID 4688 com linha de comando aparece no Event Viewer ao abrir um app

## Commit
```bash
git add .
git commit -m "feat: hardening - 6 GPOs (senha, auditoria, LLMNR/SMBv1, PS logging, USB, banner)"
git push
```

Próximo: **Fase 3 — Monitoramento (Wazuh + Sysmon)**.
