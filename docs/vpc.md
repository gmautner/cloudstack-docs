# VPC

## Introdução

Os diagramas abaixo resumem as principais diferenças entre _Guest Networks_, conceito que viemos utilizando até aqui, e _VPC_, que introduziremos nesta seção.

### Guest Network

```mermaid
graph LR
    subgraph "Internet Pública"
        Internet(Internet)
    end

    subgraph "Guest Network 10.0.0.0/8"
        A1(Web 1<br>10.1.1.2)
        A2(Web 2<br>10.1.1.3)
        A3(App<br>10.1.1.4)
        A4(DB<br>10.1.1.5)
    end
    
    Internet --> RD(VR:<br>NAT<br>Load Balancing<br>Firewall<br>Gateway: 10.1.1.1)
    
    RD --> A1
    RD --> A2
    RD --> A3
    RD --> A4

    style RD fill:#cc9999
```

### VPC

```mermaid
graph LR
    subgraph "Internet Pública"
        Internet(Internet)
    end

    subgraph "VPC 10.0.0.0/8"
        subgraph "BD tier 10.0.3.0/24"
            DB(DB<br>10.1.3.10)
        end

        subgraph "App tier 10.0.2.0/24"
            App(App<br>10.1.2.50)
        end

        subgraph "Web tier 10.0.1.0/24"
            Web1(Web 1<br>10.1.1.101)
            Web2(Web 2<br>10.1.1.102)
        end

    end

    Internet --> CS_VR(VR:<br>NAT<br>Load Balancing<br>ACL<br>Gateway: 10.1.1.1<br>Gateway: 10.1.2.1<br>Gateway: 10.1.3.1)

    CS_VR --> Web1
    CS_VR --> Web2
    CS_VR --> App
    CS_VR --> DB

    style CS_VR fill:#cc9999
```

Em resumo:

- Enquanto numa _Guest Network_ há apenas um segmento de rede, com visibilidade irrestrita entre as _VMs_, numa _VPC_ é possível segmentar a rede entre diferentes _tiers_.
- Na _Guest Network_ utilizamos _Firewalls_ para cada IP. Na _VPC_ as regras de acesso são definidas via _ACLs_ (_Access Control Lists_) entre as _tiers_ e entre estas e a internet pública.
  
Neste tutorial criaremos um ambiente com duas _tiers_, _web_ e _bd_. Resumo dos passos:

- Criar _VPC_
- Criar _ACLs_
- Criar _tiers_ alocando as respectivas _ACLs_
- Criar instâncias em cada _tier_
- Mapear IPs públicos às instâncias
- Criar _load balancer_ e _autoscaling group_

## Criar VPC

Acesse __Network__, __VPC__, __Add VPC +__ preenchendo:

- __Name__: _minha-vpc_
- __Description__: _Minha VPC_
- __CIDR__: _10.0.0.0/8_

![VPC](vpc.png)

## Criar ACLs

Em __Network__, __VPC__ clique sobre _minha-vpc_, __Network ACL lists__ e __Add network ACL list__.

Crie duas _ACL lists_ com nomes _web_ e _bd_. Descrições podem ser iguais aos nomes.

### ACL para Web

Clique sobre a _ACL_ _web_:
![ACL](acl.png)

## Criar Tiers

Em __Network__, __VPC__ clique sobre _minha-vpc_, __Networks__ e __+ Add new tier__.

Preencha com:

