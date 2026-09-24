# Passo a passo — VMware (redes) + pfSense (FW01)

> Status: ✅ **Concluído.** Mantido como referência e para reprodutibilidade.

---

## Parte A — Redes virtuais no VMware

**Edit → Virtual Network Editor** (como administrador). Criar quatro redes **Host-only**:

| VMnet | Zona | Configuração |
|---|---|---|
| VMnet10 | Servidores | Host-only, **sem** DHCP do VMware, **sem** adaptador do host |
| VMnet11 | Usuários | idem |
| VMnet12 | Segurança | idem |
| VMnet13 | Ataque | idem |

- ❌ Desmarcar **"Use local DHCP service"** → quem distribui IP é a empresa (o DC), não o VMware.
- ❌ Desmarcar **"Connect a host virtual adapter"** → mantém o PC real fora das redes do lab (isolamento).

---

## Parte B — Criar a VM do pfSense (FW01)

1. **File → New Virtual Machine → Custom**
2. Guest OS: **Other → FreeBSD 14 64-bit**
3. Nome: `FW01` · 1 CPU / 2 núcleos · 2 GB RAM · disco 20 GB
4. **Customize Hardware → Add → Network Adapter** até ter **5 placas**, nesta ordem:

| Placa | Conexão | Vira |
|---|---|---|
| Adapter 1 | NAT (VMnet8) | WAN |
| Adapter 2 | Custom: VMnet10 | LAN (Servidores) |
| Adapter 3 | Custom: VMnet11 | OPT1 (Usuários) |
| Adapter 4 | Custom: VMnet12 | OPT2 (Segurança) |
| Adapter 5 | Custom: VMnet13 | OPT3 (Ataque) |

> 💡 **Anote o MAC de cada placa** (Advanced). É como você confere qual VMnet corresponde a qual interface do pfSense. Pular isso = interfaces trocadas (aconteceu — ver 07-lessons-learned).

---

## Parte C — Instalação do pfSense

1. Baixar o **Netgate Installer** (ISO, AMD64) em pfsense.org/download → extrair o `.gz` com 7-Zip.
2. Apontar o CD/DVD da VM para a ISO e ligar.
3. Aceitar a licença → **Install** → **Install CE** (não validar assinatura Plus).
4. Sistema de arquivos: **ZFS**, esquema **GPT**, dispositivo **Stripe**.
5. Selecionar o disco de 20 GB e confirmar.
6. Escolher a última versão estável → aguardar → **Reboot**.
7. Ao desligar, **desmarcar "Connect at power on"** no CD/DVD (senão reinicia o instalador).

📸 Snapshot: `FW01-00-instalacao-limpa`

---

## Parte D — Atribuir interfaces e IPs (console)

**Opção 1 — Assign Interfaces** (usar os MACs anotados):
- VLANs? → **n**
- WAN → placa 1 · LAN → placa 2 · OPT1 → placa 3 · OPT2 → placa 4 · OPT3 → placa 5

**Opção 2 — Set interface IP address** (repetir por interface):

| Interface | IPv4 | Máscara | Gateway | DHCP |
|---|---|---|---|---|
| WAN | via DHCP | — | (do NAT) | — |
| LAN | 192.168.10.1 | 24 | nenhum | não |
| OPT1 | 192.168.20.1 | 24 | nenhum | não |
| OPT2 | 192.168.30.1 | 24 | nenhum | não |
| OPT3 | 192.168.99.1 | 24 | nenhum | não |

- "Revert to HTTP?" → **n** (manter HTTPS).
- Gateway em branco nas internas porque **o pfSense é o gateway** delas.
- DHCP desligado porque quem distribui IP é o SRV-DC01 (via relay).

📸 Snapshot: `FW01-01-interfaces-configuradas`

---

## Provas

- [x] Console mostra WAN com IP e LAN/OPT1/OPT2/OPT3 corretos
- [x] Console opção 7 (Ping) → `8.8.8.8` responde (WAN tem internet)

## Resultado obtido

pfSense **2.9.0-RELEASE** instalado, 5 interfaces configuradas, WAN via DHCP, ping externo OK.
