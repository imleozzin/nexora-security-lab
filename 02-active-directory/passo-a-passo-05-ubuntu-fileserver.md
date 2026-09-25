# Passo a passo — Ubuntu SRV-FILE01 (servidor de arquivos) — fecha a Fase 1

> Status: ⬜ A executar. Último item da Fase 1.
> Objetivo: subir um servidor de arquivos Linux com Samba, integrado ao domínio, com pastas por departamento (base do RBAC).

---

## Etapa 1 — Criar a VM
- Ubuntu Server 24.04 LTS, 1–2 vCPU, 2 GB RAM, disco 40 GB, rede **VMnet10 (Servidores)**.
- Durante a instalação: hostname `srv-file01`, criar usuário admin, marcar **Install OpenSSH server**.

## Etapa 2 — Rede estática
Edite o netplan:
```bash
sudo nano /etc/netplan/00-installer-config.yaml
```
```yaml
network:
  version: 2
  ethernets:
    ens33:
      dhcp4: no
      addresses: [192.168.10.20/24]
      routes:
        - to: default
          via: 192.168.10.1
      nameservers:
        addresses: [192.168.10.10]   # o DC é o DNS
        search: [nexora.local]
```
```bash
sudo netplan apply
ip a                      # confirma 192.168.10.20
ping -c2 192.168.10.10    # alcança o DC
ping -c2 8.8.8.8          # tem internet (via pfSense)
```

## Etapa 3 — Atualizar e instalar Samba
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y samba
```

## Etapa 4 — Criar as pastas por departamento
```bash
sudo mkdir -p /srv/compartilhado/{RH,Financeiro,TI,Comercial,Diretoria}
sudo chmod -R 770 /srv/compartilhado
```

## Etapa 5 — Configurar o Samba
```bash
sudo nano /etc/samba/smb.conf
```
No fim do arquivo, um bloco por pasta (exemplo RH):
```ini
[RH]
   path = /srv/compartilhado/RH
   browseable = yes
   read only = no
   valid users = @"NEXORA\GRP-RH"
```
Repita para Financeiro, TI, Comercial, Diretoria (trocando nome e grupo).

```bash
sudo smbcli testparm      # valida a sintaxe (ou: testparm)
sudo systemctl restart smbd
sudo systemctl enable smbd
```

## Etapa 6 — Testar do WS-RH01
No Windows Explorer: `\\192.168.10.20` → deve pedir credencial → logado como `nexora\maria.silva` (do RH), ela **acessa a pasta RH** mas **não** a Financeiro. Isso é RBAC na prática.

> Nota: integração completa do Linux ao AD (Kerberos via `realmd`/`sssd`) é um upgrade opcional. Para o lab, o mapeamento por grupo no smb.conf já demonstra o conceito. Documente como melhoria futura se não fizer o join completo agora.

---

## Provas
- [ ] `ip a` mostrando 192.168.10.20
- [ ] `\\192.168.10.20` acessível do WS-RH01
- [ ] Usuário do RH acessa só a pasta RH (print do "acesso negado" na Financeiro é ótima evidência)

## 🎉 Fase 1 concluída
Com isso a Fase 1 (Construir) fica 100%. Próximo: **Fase 2 — Hardening + GPOs** (`03-hardening/`).

## Commit
```bash
git add .
git commit -m "feat: Ubuntu SRV-FILE01 com Samba por departamento - Fase 1 completa"
git push
```
