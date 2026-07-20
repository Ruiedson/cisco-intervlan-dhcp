# 🌐 Projeto Cisco Packet Tracer: VLANs, Inter-VLAN Routing, SVIs & DHCP Dynamic Allocation

Este projeto demonstra a implementação de uma infraestrutura de rede corporativa segmentada utilizando o **Cisco Packet Tracer**. A topologia abrange o dimensionamento de sub-redes com **VLSM**, isolamento de tráfego por **VLANs**, roteamento inter-VLAN via **SVIs (Switch Virtual Interfaces)** em um Switch Layer 3 (Multicamada) e atribuição dinâmica de endereços com **Servidor DHCP**.

---

## 📐 Topologia do Projeto

![Topologia da Rede](topologia.png)

---

## 📑 Tabela de Endereçamento e Sub-redes (VLSM)

A rede foi dividida em três sub-redes para otimizar o espaço de endereçamento IP e garantir o isolamento entre departamentos:

| VLAN ID | Nome | Sub-rede / CIDR | Máscara de Rede | Gateway Padrão (SVI) | Alcance de IPs Úteis |
| :---: | :--- | :--- | :--- | :--- | :--- |
| **VLAN 10** | **VENDAS** | `192.168.10.0/28` | `255.255.255.240` | `192.168.10.1` | `192.168.10.2` – `192.168.10.14` |
| **VLAN 20** | **R.HUMANOS** | `192.168.10.16/28` | `255.255.255.240` | `192.168.10.17` | `192.168.10.18` – `192.168.10.30` |
| **VLAN 30** | **IT** | `192.168.10.32/29` | `255.255.255.248` | `192.168.10.33` | `192.168.10.34` – `192.168.10.38` |

---

## 🎯 Requisitos Implementados

1. **Hostnames e Organização:** Nomeação padronizada dos ativos de rede (`TECH` para o Core e `SW1` para o Acesso).
2. **Segurança do Ativo:** 
   * Senha do modo privilegiado ativada (`enable secret`).
   * Proteção de acesso às linhas de Console e VTY (Telnet/SSH).
3. **Banner de Advertência:** `banner motd` configurado avisando sobre acesso restrito.
4. **Segregação por VLANs:** Portas dos switches atribuídas rigorosamente conforme a divisão por setores.
5. **Roteamento Inter-VLAN (SVIs):** Habilitação do recurso `ip routing` no switch `TECH` e configuração de Interfaces Virtuais (VLAN 10, 20 e 30) como gateways.
6. **Atribuição Dinâmica (DHCP):**
   * Exclusão dos IPs dos gateways do pool (`ip dhcp excluded-address`).
   * Escopos DHCP configurados para distribuir IP, Máscara e Default Gateway automaticamente para todas as máquinas.

---

## 💻 Resumo das Configurações CLI (Cisco IOS)

<details>
<summary><b>🔍 Clique para expandir as configurações do Switch Core (TECH)</b></summary>

```cisco
enable
configure terminal
hostname TECH

! Segurança e Banners
enable secret 123
banner motd ^CACCESSO RESTRITO, APENAS PESSOAL AUTORIZADO^C

line con 0
 password 123
 login
exit
line vty 0 15
 password 123
 login
exit

! Roteamento e VLANs
ip routing

vlan 10
 name VENDAS
vlan 20
 name R.HUMANOS
vlan 30
 name IT
exit

! Atribuição de Portas - TECH
interface range FastEthernet0/1 - 4
 switchport mode access
 switchport access vlan 30
exit

interface range FastEthernet0/5 - 8
 switchport mode access
 switchport access vlan 10
exit

interface range FastEthernet0/9 - 11
 switchport mode access
 switchport access vlan 20
exit

! Tronco (Trunk) para o SW1
interface range GigabitEthernet0/1 - 2
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit

! SVIs (Gateways)
interface Vlan10
 ip address 192.168.10.1 255.255.255.240
 no shutdown
exit

interface Vlan20
 ip address 192.168.10.17 255.255.255.240
 no shutdown
exit

interface Vlan30
 ip address 192.168.10.33 255.255.255.248
 no shutdown
exit

! Serviços DHCP
ip dhcp excluded-address 192.168.10.1
ip dhcp excluded-address 192.168.10.17
ip dhcp excluded-address 192.168.10.33

ip dhcp pool VENDAS
 network 192.168.10.0 255.255.255.240
 default-router 192.168.10.1
exit

ip dhcp pool R.HUMANOS
 network 192.168.10.16 255.255.255.240
 default-router 192.168.10.17
exit

ip dhcp pool IT
 network 192.168.10.32 255.255.255.248
 default-router 192.168.10.33
exit

enable
configure terminal
hostname SW1

! Segurança e Banners
enable secret 123
banner motd ^CACESSO RESTRITO, APENAS AUTORIZADO^C

line con 0
 password 123
 login
exit
line vty 0 15
 password 123
 login
exit

! Criação das VLANs no Banco
vlan 10
 name VENDAS
vlan 20
 name R.HUMANOS
vlan 30
 name IT
exit

! Atribuição de Portas - SW1
interface range FastEthernet0/1 - 4
 switchport mode access
 switchport access vlan 10
exit

interface range FastEthernet0/5 - 6
 switchport mode access
 switchport access vlan 30
exit

interface range FastEthernet0/7 - 10
 switchport mode access
 switchport access vlan 20
exit

! Tronco (Trunk) para o TECH
interface range GigabitEthernet0/1 - 2
 switchport mode trunk
exit