# FortiOS · cheatsheet de troubleshooting

Comandos do dia a dia. Onde aparecer `<algo>`, troque pelo valor real. Os exemplos ficam sempre abaixo do bloco, para o copiar pegar só o comando.

> **Parar qualquer debug** (guarde esse):
> ```fortios
> diagnose debug disable
> diagnose debug reset
> ```

## Index

- [Iprope Lookup](#iprope-lookup)
- [System and Status](#system-and-status)
- [Routing](#routing)
- [ARP](#arp)
- [Sessions](#sessions)
- [Ping / Traceroute / Telnet](#ping--traceroute--telnet)
- [Packet Sniffer](#packet-sniffer)
- [Debug Flow](#debug-flow)
- [IPSec VPN (Site-to-Site)](#ipsec-vpn-site-to-site)
- [SSL-VPN](#ssl-vpn)
- [SAML](#saml)
- [SSL User Authentication](#ssl-user-authentication)
- [FSSO](#fsso)
- [DHCP](#dhcp)
- [Web Filter](#web-filter)
- [BGP](#bgp)
- [HA (High Availability)](#ha-high-availability)
- [Link Monitor / SD-WAN](#link-monitor--sd-wan)
- [Firewall Policy](#firewall-policy)
- [Automation Stitch](#automation-stitch)
- [Interfaces and Hardware](#interfaces-and-hardware)
- [REST API](#rest-api)
- [Sudo (Cross-VDOM Commands)](#sudo-cross-vdom-commands)

---

## Iprope Lookup

**Testar qual política casa com um tráfego, sem gerar tráfego**
```fortios
diagnose firewall iprope lookup <ip-origem> <porta-origem> <ip-destino> <porta-destino> <protocolo> <interface-origem>
```
Ex.: `diagnose firewall iprope lookup 10.210.14.139 0 57.144.179.32 443 6 port3`

| # | Campo | Observação |
|---|---|---|
| 1 | IP de origem | |
| 2 | Porta de origem | Use `0` quando não importar |
| 3 | IP de destino | |
| 4 | Porta de destino | |
| 5 | Protocolo | Número, ver tabela abaixo |
| 6 | Interface de origem | Por onde o tráfego entra (porta, VLAN ou túnel) |

| ICMP | TCP | UDP |
|:---:|:---:|:---:|
| 1 | 6 | 17 |

---

## System and Status

**Status geral**
```fortios
get system status
```

**Informações do sistema**
```fortios
get system info
```

**Status do HA**
```fortios
get system ha status
```

**Configuração global**
```fortios
get system global
```

**Mostrar grupos de perfil de segurança na GUI**
```fortios
config system settings
    set gui-security-profile-group enable
end
```

**Ler o crashlog**
```fortios
diagnose debug crashlog read
```

**Ler o log de erros de configuração** (bom checar depois de upgrade)
```fortios
diagnose debug config-error-log read
```

**Resumo de CPU e memória**
```fortios
diagnose sys top-summary
```

**CPU e memória por processo** (conserve mode)

Atualiza a cada 5 s, 30 vezes. Dentro do `top`: `c` ordena por CPU, `m` por memória, `q` sai.
```fortios
diagnose sys top 5 30
```

Mesma coisa, mostrando só os 3 primeiros processos:
```fortios
diagnose sys top 5 30 3
```

Memória:
```fortios
get hardware memory
diagnose sys top-mem 10
diagnose sys top-mem detail
```

**Informações da placa de rede** (velocidade, duplex, erros)
```fortios
diagnose hardware deviceinfo nic <porta>
```
Ex.: `diagnose hardware deviceinfo nic lan1`

**Teste de velocidade da interface**
```fortios
diagnose traffictest run
```

---

## Routing

**Qual rota o firewall usa para um IP**
```fortios
get router info routing-table details <ip>
```
Ex.: `get router info routing-table details 192.168.1.99`

**Resumo do BGP**
```fortios
get router info bgp summary
```

**Limpar sessão BGP (soft, entrada)** — reaprende as rotas sem derrubar o vizinho
```fortios
execute router clear bgp ip <ip-vizinho> soft in
```
Ex.: `execute router clear bgp ip 10.2.0.250 soft in`

---

## ARP

**Tabela ARP completa**
```fortios
get system arp
```

**Filtrar por IP ou range**
```fortios
get system arp | grep <ip-ou-rede>
```
Ex.: `get system arp | grep 10.11.100.`

**Procurar um IP na configuração**
```fortios
show | grep <ip>
```

---

## Sessions

**Listar sessões ativas**
```fortios
diagnose sys session list
```

**Limpar todas as sessões**
```fortios
diagnose sys session clear
```
> ⚠ Sem filtro, derruba **todas** as sessões do firewall.

**Apagar uma sessão pelo ID**
```fortios
diagnose sys session delete <id-da-sessão>
```

**Filtrar por IP de destino antes de limpar** (aceita um IP ou uma faixa)
```fortios
diagnose sys session filter clear
diagnose sys session filter dst <ip-inicial> <ip-final>
diagnose sys session filter
diagnose sys session clear
```
Ex.: `diagnose sys session filter dst 10.3.79.165 10.3.79.185`

O `filter` sem argumento mostra o filtro aplicado, para conferir antes do `clear`.

**Filtrar por IP de origem antes de limpar**
```fortios
diagnose sys session filter clear
diagnose sys session filter src <ip-inicial> <ip-final>
diagnose sys session filter
diagnose sys session clear
```
Ex.: `diagnose sys session filter src 10.3.79.165 10.3.79.185`

---

## Ping / Traceroute / Telnet

**Ping simples**
```fortios
execute ping <ip>
```

**Ping com IP de origem específico** — precisa ser IP de uma interface do firewall
```fortios
execute ping-options source <ip-interface>
execute ping <ip-destino>
```
Ex.: `execute ping-options source 10.11.100.232`

Para voltar ao padrão:
```fortios
execute ping-options reset
```

**Traceroute**
```fortios
execute traceroute <ip>
```

**Telnet (testar porta)**
```fortios
execute telnet <ip> <porta>
```
Ex.: `execute telnet 10.11.100.232 22`

---

## Packet Sniffer

Formato: `diagnose sniffer packet <interface> "<filtro>" <verbosidade> [qtd] [tempo]`. Verbosidade `4` mostra a interface de entrada e saída. `Ctrl+C` para a captura.

**Por host**
```fortios
diagnose sniffer packet any "host <ip>" 4
```
Ex.: `diagnose sniffer packet any "host 10.211.12.68" 4`

**Host + protocolo (ICMP)**
```fortios
diagnose sniffer packet any "host <ip> and icmp" 4
```
Ex.: `diagnose sniffer packet any "host 201.55.34.141 and icmp" 4`

**Origem e destino específicos**
```fortios
diagnose sniffer packet any "host <ip-a> and host <ip-b>" 4
```
Ex.: `diagnose sniffer packet any "host 201.55.34.141 and host 10.211.12.68" 4`

---

## Debug Flow

Mostra o caminho do pacote dentro do firewall: rota escolhida, política que bateu, NAT e motivo de drop.

**Debug flow básico**
```fortios
diagnose debug disable
diagnose debug flow trace stop
diagnose debug flow filter clear
diagnose debug flow filter saddr <ip-origem>
diagnose debug flow filter daddr <ip-destino>
diagnose debug flow filter dport <porta>
diagnose debug flow show console enable
diagnose debug flow show function-name enable
diagnose debug flow show iprope enable
diagnose debug console timestamp enable
diagnose debug flow trace start 9999
diagnose debug enable
```

**Filtros disponíveis** (`diagnose debug flow filter ?`)

| Filtro | O que faz | Exemplo |
|---|---|---|
| `clear` | Limpa todos os filtros | `filter clear` |
| `vd` | VDOM pelo índice | `filter vd 0` |
| `vd-name` | VDOM pelo nome | `filter vd-name root` |
| `proto` | Protocolo (1 ICMP, 6 TCP, 17 UDP) | `filter proto 6` |
| `addr` | IP de origem **ou** destino (aceita faixa) | `filter addr 10.2.54.3 10.2.35.5` |
| `saddr` | IP de origem | `filter saddr 10.255.254.1` |
| `daddr` | IP de destino | `filter daddr 8.8.8.8` |
| `port` | Porta de origem **ou** destino | `filter port 5060` |
| `sport` | Porta de origem | `filter sport 5060` |
| `dport` | Porta de destino | `filter dport 1521` |
| `negate` | Inverte o filtro seguinte | `filter negate saddr 10.44.114.36` |

> Todos começam com `diagnose debug flow`. Rodar o filtro sem valor (ex.: `filter saddr`) remove só aquele filtro.

**Parar o debug**
```fortios
diagnose debug disable
```

**Flow completo com iprope (lookup de rota e política)**
```fortios
diagnose debug disable
diagnose debug flow trace stop
diagnose debug flow filter clear
diagnose debug flow show iprope enable
diagnose debug flow filter dport <porta>
diagnose debug flow show console enable
diagnose debug flow show function-name enable
diagnose debug console timestamp enable
diagnose debug flow trace start 999999
diagnose debug enable
```
Ex.: `diagnose debug flow filter dport 514`

Para testar a política sem gerar tráfego, ver [Iprope Lookup](#iprope-lookup).

**Ver interfaces via fnsysctl**
```fortios
fnsysctl ifconfig <interface>
```

**Listar os IPs do firewall**
```fortios
diagnose ip address list
```

**Ver zonas com VLANs**
```fortios
show system zone | grep VLAN
```

---

## IPSec VPN (Site-to-Site)

**Debug da negociação IKE**
```fortios
diagnose debug disable
diagnose debug reset
diagnose vpn ike log-filter clear
diagnose vpn ike log-filter dst-addr4 <ip-local>
diagnose vpn ike log-filter rem-addr4 <ip-peer>
diagnose debug application ike -1
diagnose debug console timestamp enable
diagnose debug enable
```

**Ver se a fase 1 subiu**
```fortios
diagnose vpn ike gateway list name "<nome-vpn>"
```

**Resumo do túnel (fase 1 e 2)**
```fortios
get vpn ipsec tunnel summary | grep -i -f "<nome-vpn>"
```

---

## SSL-VPN

**Debug geral**
```fortios
diagnose debug reset
diagnose debug application fnbamd 255
diagnose debug enable
```

**Listar e derrubar sessões**
```fortios
execute vpn sslvpn list
execute vpn sslvpn del-all
```

---

## SAML

**Debug de autenticação SAML**
```fortios
diagnose debug application httpsd -1
diagnose debug application samld -1
diagnose debug console timestamp enable
diagnose debug enable
```

---

## SSL User Authentication

**Padrão (usuário e senha locais)**
```fortios
diagnose debug reset
diagnose debug console timestamp enable
diagnose debug application fnbamd -1
diagnose debug application authd -1
diagnose debug application sslvpn -1
diagnose debug enable
```

**Com SAML**
```fortios
diagnose vpn ssl debug-filter src-addr4 <ip-usuario>
diagnose debug application samld -1
diagnose debug application sslvpn -1
diagnose debug enable
```

---

## FSSO

**Status do servidor FSSO**
```fortios
diagnose debug enable
diagnose debug authd fsso server-status
```

---

## DHCP

**Debug do serviço DHCP**
```fortios
diagnose debug reset
diagnose debug application dhcps -1
diagnose debug enable
```

**Parar o debug**
```fortios
diagnose debug reset
diagnose debug disable
```

---

## Web Filter

**Testar conectividade com o FortiGuard**
```fortios
diagnose debug rating
execute ping service.fortiguard.net
execute ping update.fortiguard.net
execute ping guard.fortinet.net
```

**Ver categoria de web filter nos logs**
```fortios
execute log filter category utm-webfilter
execute log display
```

**Debug de rating e atualização de base**
```fortios
diagnose debug application update -1
diagnose debug enable
execute update-now
```

**Procurar FQDN nos objetos do firewall**
```fortios
diagnose firewall fqdn list | grep -i -f <dominio>
```
Ex.: `diagnose firewall fqdn list | grep -i -f youtube`

**Ver os perfis de web filter**
```fortios
show webfilter profile
```

**Criar tabela de URL filter**

O `edit 0` pega o próximo ID livre. Use `set type wildcard` quando a URL tiver `*`; URL exata usa o tipo padrão.
```fortios
config webfilter urlfilter
    edit 0
        set name "<nome-da-tabela>"
        config entries
            edit 1
                set url "<url>"
                set type wildcard
            next
        end
    next
end
```

Confira o ID gerado:
```fortios
show webfilter urlfilter
```

**Ligar a tabela no perfil**
```fortios
config webfilter profile
    edit "<perfil>"
        config web
            set urlfilter-table <id-da-tabela>
        end
    next
end
```

Exemplo (tabela RAVPN-WEB-PROF):
```fortios
config webfilter urlfilter
    edit 0
        set name "RAVPN-WEB-PROF"
        config entries
            edit 1
                set url "*.qualys.com"
                set type wildcard
            next
            edit 2
                set url "*.santandergatewayft.com.br"
                set type wildcard
            next
            edit 3
                set url "download.docker.com"
                set type wildcard
            next
            edit 4
                set url "*.redhat.com"
                set type wildcard
            next
            edit 5
                set url "reinf.receita.fazenda.gov.br"
            next
            edit 6
                set url "preprodefdreinf.receita.fazenda.gov.br"
            next
            edit 7
                set url "maps.googleapis.com"
                set type wildcard
            next
            edit 8
                set url "totvs.logistica.totvs.app"
                set type wildcard
            next
            edit 9
                set url "gefcoindustria.logistica.totvs.app"
                set type wildcard
            next
            edit 10
                set url "admin.rac.totvs.app"
                set type wildcard
            next
            edit 11
                set url "40.121.71.32"
            next
            edit 12
                set url "*argoit.com.br*"
                set type wildcard
            next
            edit 14
                set url "*api.unidas.com.br*"
                set type wildcard
            next
            edit 15
                set url "*multirio.com.br*"
                set type wildcard
            next
        end
    next
end
```

---

## BGP

Ver a seção [Routing](#routing).

---

## HA (High Availability)

**Acessar um membro do cluster**
```fortios
execute ha manage <id-membro> <usuario>
```
Ex.: `execute ha manage 0 jusantos`

**Definir prioridade do nó**
```fortios
execute ha set-priority <serial> <prioridade>
```
Ex.: `execute ha set-priority FG4H0E5819900156 151`

**Zerar uptime do HA (forçar nova eleição)**
```fortios
diagnose sys ha reset-uptime
```
> ⚠ Pode causar failover. Faça em janela.

---

## Link Monitor / SD-WAN

**Criar probe de link monitor**

`gateway-ip` é o next-hop; `server` é o IP que vai ser testado.
```fortios
config system link-monitor
    edit "<nome-probe>"
        set srcintf <interface>
        set gateway-ip <next-hop>
        set server <ip-alvo>
    next
end
```

Exemplo:
```fortios
config system link-monitor
    edit "ProbeMPLS"
        set srcintf port1
        set gateway-ip 10.112.36.194
        set server 10.41.122.2
    next
end
```

---

## Firewall Policy

**Mover uma regra**
```fortios
config firewall policy
    move <id> before <id-referencia>
end
```
Ex.: `move 956 before 905`

---

## Automation Stitch

**Debug de automation que não dispara**
```fortios
diagnose debug reset
diagnose debug application autod -1
diagnose debug enable
```
> Referência: [Fortinet Community: how to check why automation stitch is not working](https://community.fortinet.com/fortigate-3/technical-tip-how-to-check-why-automation-stitch-is-not-working-as-expected-92733)

---

## Interfaces and Hardware

Ver [System and Status](#system-and-status) para informações da placa de rede e teste de velocidade.

---

## REST API

**Ver a PSK de uma VPN IPsec em texto puro (phase1-interface)**

Precisa estar logado na GUI no mesmo navegador.
```
https://<ip-firewall>:<porta-admin>/api/v2/cmdb/vpn.ipsec/phase1-interface/<nome-vpn>?plain-text-password=1
```

Exemplos:
```
https://192.168.253.253:11443/api/v2/cmdb/vpn.ipsec/phase1-interface/VPN_DC-01?plain-text-password=1
https://192.168.1.99:11443/api/v2/cmdb/vpn.ipsec/phase1-interface/VPN_DC-WCS?plain-text-password=1
```

---

## Sudo (Cross-VDOM Commands)

Roda um comando em outra VDOM sem precisar entrar nela.
```fortios
sudo <vdom> get router info routing-table details <ip>
```
Ex.: `sudo FILIAIS get router info routing-table details 10.0.0.1`
