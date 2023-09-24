# Introdução

A ideia deste guia é conhecer o CloudStack através de um exemplo de uma aplicação Laravel com banco de dados MySQL.

Os conceitos servem para qualquer outro framework (Ruby, Python etc.), qualquer outro banco (Postgres, Mongo) e qualquer tipo de arquitetura (monolito, microsserviços)

## Principais conceitos

Ao final deste tutorial você terá total domínio de como fazer:

- Instâncias
- Volumes
- Chaves SSH
- Templates
- Redes
- IPs
- Firewalls
- Load balancers
- Grupos de autoscaling

## Vantagens do CloudStack

### Open Source

O CloudStack é um projeto __100% open source__ da _Apache Foundation_, eliminando assim a dependência de cloud providers e do uso de ferramentas proprietárias que implicam em _alto lockin_ e _forte acomplamento_ entre aplicações e infraestrutura.

### Custos

Para as empresas do grupo, a estimativa de economia de custos é de:

- Redução de __47%__ [^1] a __80%__ [^2] sobre equivalente AWS (Ice Lake) em Ohio, 1 Year Reserved No Upfront, sem impostos.

- Redução de __67%__ [^1] a __87%__ [^2] sobre equivalente AWS (Ice Lake) em South America, 1 Year Reserved No Upfront, sem impostos.

- Redução de __50%__ de storage sobre AWS (gp3) Ohio e __73%__ sobre South America

- Redução de __84%__ de banda sobre Ohio e __93%__ sobre South America

[^1]: 64 cores/512 GB RAM
[^2]: 64 cores/128 GB RAM