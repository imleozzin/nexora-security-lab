# Decisões de Arquitetura (ADR)

> Esta é a seção que um recrutador técnico realmente lê. Não descreve **o que** foi feito (isso está no resto do repo), mas **por que**.
>
> ✍️ **Preencha com suas próprias palavras.** Os campos abaixo estão em branco de propósito. Explique como se estivesse respondendo a um entrevistador. Há dicas em cada um.

---

## ADR-01 — Por que segmentar a rede em zonas?

**Decisão:** separar servidores, usuários, segurança e ataque em redes distintas roteadas por firewall.

**Justificativa:**
> _(escreva aqui)_

<!-- Dica: fale sobre reduzir o "raio de explosão" (blast radius) de um comprometimento,
     aplicar menor privilégio no nível de rede, e forçar o tráfego entre zonas a passar
     pelo firewall (onde vira log). -->

---

## ADR-02 — Por que a zona de Ataque (Kali) é isolada e sem internet?

**Decisão:** o Kali fica em rede própria, sem rota para a internet, liberado só durante testes.

**Justificativa:**
> _(escreva aqui — 2 frases)_

<!-- Pontos que a resposta forte cobre:
     - Contenção: ferramentas de ataque/malware de teste não podem vazar para a rede real
       nem para a internet (é também questão legal — atacar o que não é seu é crime).
     - Controle: o ataque só ocorre quando VOCÊ libera a regra no pfSense.
     - Visibilidade: o tráfego do atacante passa pelo firewall, então gera log — e é esse
       log que você investiga.
     Vocabulário: containment, attack surface. -->

---

## ADR-03 — Por que o SIEM fica isolado dos usuários?

**Decisão:** o SRV-SIEM fica na zona de Segurança, aceitando apenas as portas do agente (1514/1515).

**Justificativa:**
> _(escreva aqui)_

<!-- Pontos-chave:
     - O SIEM guarda as PROVAS. Após comprometer um PC, o atacante tenta chegar ao SIEM
       para apagar logs / desligar o monitoramento (MITRE ATT&CK: T1070 Indicator Removal,
       T1562 Impair Defenses).
     - Isolar reduz a superfície de ataque e protege a integridade das evidências.
     Vocabulário: log integrity, defense evasion. -->

---

## ADR-04 — Por que a WAN usa DHCP e as interfaces internas usam IP fixo?

**Decisão:** WAN recebe IP dinâmico; LAN/OPT1/OPT2/OPT3 têm IP estático.

**Justificativa:**
> _(escreva aqui, usando a palavra "gateway")_

<!-- Pontos-chave:
     - O pfSense não controla a rede de fora; quem define o IP da WAN é o provedor
       (no lab, o NAT do VMware). Se mudar, ninguém interno depende disso.
     - O pfSense É o gateway de cada zona. Clientes, escopo DHCP e regras apontam para
       192.168.X.1. Se esse IP mudasse, a zona perderia a rota. Infraestrutura precisa
       de endereço previsível. -->

---

## ADR-05 — Por que o DNS do DC aponta para 127.0.0.1 (loopback)?

**Decisão:** após a promoção, o SRV-DC01 usa a si mesmo como DNS primário.

**Justificativa:**
> _(escreva aqui)_

<!-- Pontos-chave:
     - Um DC é também servidor DNS da floresta; ele deve consultar o próprio serviço para
       resolver os registros do domínio (SRV records de Kerberos/LDAP etc.).
     - A internet é resolvida via forwarder para o pfSense.
     - ARMADILHA que você viveu: apontar para loopback ANTES de o serviço DNS existir
       (antes da promoção) deixa o servidor sem resolver nomes. Ver 07-lessons-learned. -->

---

## ADR-06 — Decisões temporárias assumidas (dívida técnica do lab)

Registrar aqui o que foi feito de forma não ideal por conveniência, para corrigir depois.

- **Gestão do pfSense a partir do DC:** em produção, a administração deveria vir da zona de Segurança, não de um Domain Controller. Corrigir na Fase 2.
- **Regras "allow any" em Usuários/Segurança:** temporárias, para destravar a construção. Endurecer na Fase 2.
- _(adicione outras conforme surgirem)_
