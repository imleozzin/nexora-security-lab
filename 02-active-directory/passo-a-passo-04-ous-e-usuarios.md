# Passo a passo — OUs, usuários e organização do AD (fecha a Fase 1)

> Status: ⬜ A executar no SRV-DC01. Fecha a parte de Active Directory da Fase 1.
>
> Objetivo: criar a estrutura organizacional da Nexora (OUs por departamento), usuários e grupos, e arrumar o WS-RH01 no lugar certo.

---

## Etapa 1 — Criar a estrutura de OUs e grupos

O script já está no repo: `02-active-directory/scripts/criar-estrutura-ad.ps1`.

No SRV-DC01, PowerShell como administrador:
```powershell
cd <pasta-do-repo>\02-active-directory\scripts
.\criar-estrutura-ad.ps1
```

Se der erro de política de execução:
```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```
(vale só para esta sessão — não deixa a máquina permissiva de forma permanente).

**Confirme no ADUC** (`dsa.msc`) ou por PowerShell:
```powershell
Get-ADOrganizationalUnit -Filter * | Select Name, DistinguishedName
```
Espere ver: OU=Nexora → Usuarios (Diretoria/TI/RH/Financeiro/Comercial), Computadores (Workstations/Servidores), Grupos, Admins, ContasServico. E os grupos `GRP-*`.

---

## Etapa 2 — Criar os usuários (via CSV — pratica automação)

1. Crie o arquivo `usuarios.csv` na pasta scripts, com o cabeçalho:
```
Nome,Sobrenome,Login,Departamento
Maria,Silva,maria.silva,RH
Joao,Souza,joao.souza,Financeiro
Ana,Costa,ana.costa,TI
Carlos,Lima,carlos.lima,Comercial
Paula,Rocha,paula.rocha,Diretoria
```
(coloque ~15 no total, distribuídos pelos departamentos)

2. Descomente o bloco `Import-Csv` no fim do `criar-estrutura-ad.ps1` e rode de novo, OU rode direto:
```powershell
Import-Csv .\usuarios.csv | ForEach-Object {
    New-ADUser `
        -Name "$($_.Nome) $($_.Sobrenome)" `
        -GivenName $_.Nome -Surname $_.Sobrenome `
        -SamAccountName $_.Login `
        -UserPrincipalName "$($_.Login)@nexora.local" `
        -Path "OU=$($_.Departamento),OU=Usuarios,OU=Nexora,DC=nexora,DC=local" `
        -AccountPassword (ConvertTo-SecureString "Nexora@2026!" -AsPlainText -Force) `
        -ChangePasswordAtLogon $true -Enabled $true
    Add-ADGroupMember -Identity "GRP-$($_.Departamento)" -Members $_.Login
}
```

**Confirme:**
```powershell
Get-ADUser -Filter * -SearchBase "OU=Usuarios,OU=Nexora,DC=nexora,DC=local" | Select Name, SamAccountName
```

> ⚠️ Não commite o `usuarios.csv` com senhas reais. O `.gitignore` já bloqueia arquivos de credencial; mantenha a senha como placeholder no repo.

---

## Etapa 3 — Mover o WS-RH01 para a OU certa

Ao ingressar, o computador caiu no container padrão `CN=Computers`. Boa prática: movê-lo para `OU=Workstations`.

```powershell
$pc = Get-ADComputer -Identity "WS-RH01"
Move-ADObject -Identity $pc.DistinguishedName -TargetPath "OU=Workstations,OU=Computadores,OU=Nexora,DC=nexora,DC=local"
```

**Por quê importa:** GPOs são aplicadas por OU. Um computador no container padrão não recebe as GPOs que você vincular em Workstations. Ou seja, sem isso, o hardening da Fase 2 não chega na estação.

**Confirme:**
```powershell
Get-ADComputer WS-RH01 | Select DistinguishedName
```

---

## Etapa 4 — Testar login com usuário de departamento

No WS-RH01, faça logout e entre com `nexora\maria.silva` (senha `Nexora@2026!`, vai pedir troca no 1º login).
```powershell
whoami            # nexora\maria.silva
whoami /groups    # deve listar GRP-RH
```

📸 Print do ADUC com a árvore de OUs preenchida + `Get-ADUser` → salvar em `02-active-directory/evidencias/`.

---

## Provas
- [ ] Árvore de OUs criada (print ADUC)
- [ ] Usuários nos departamentos certos (`Get-ADUser`)
- [ ] WS-RH01 em OU=Workstations
- [ ] Login com usuário de departamento funciona

## Depois disto
Falta só o **Ubuntu SRV-FILE01** (servidor de arquivos) para a Fase 1 fechar 100%. Daí seguimos para a **Fase 2 (hardening + GPOs)** — e aí a OU=Workstations já vai estar pronta para receber as políticas.

---

## Commit sugerido
```bash
git add .
git commit -m "feat: estrutura de OUs, usuarios e grupos da Nexora"
git push
```
