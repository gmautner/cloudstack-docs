# Load Balancing e Autoscaling

Neste passo demonstraremos:

- __Load balancers__ para dividir tráfeto entre múltiplas instâncias
- __Autoscaling__ para adequar o número de instâncias a uma demanda variável

Utilizaremos os recursos criados nos tutoriais anteriores. Execute-os se ainda não o fez.

Para começar, desligaremos a instância _teste-template_ criada no passo anterior.

No menu de navegação à esquerda clique em __Compute__, __Instances__ e clique na instância _teste-template_, e clique no botão __Stop instance__.

## Criação do Load Balancer

1. No menu à esquerda selecione __Network__, __Guest networks__ e clique na rede _minha-rede_
2. Clique sobre o IP que alocamos via _Static NAT_ para _teste_template_, por exemplo `xIP_WEBx`
3. Clique em __Disable static NAT__ para liberar o IP.
4. Clique na aba __Load balancing__ e preencha conforme:
    * __Name__: _lb_
    * __Public port__: _80_
    * __Private port__: _80_
    * __CIDR list__: _0.0.0.0/0_
    * __Algorithm__: _Round-robin_
    * __Protocol__: _TCP_
    * __Autoscale__: _Yes_
    * E clique no botão __Add__
![Autoscaling](autoscaling.png)
5. Clique no botão __New__ sob __AutoScale__
![New autoscale](new-autoscale.png)

## Criação do AutoScale Group

1. Aceite a zona, e escolha o template _To Do app_
![Autoscale template](autoscale-template.png)
2. Em __Compute offering__ escolha __micro__
3. Escolha a rede _minha-rede_
![Autoscale network](autoscale-network.png)
4. Em __Load balancing__ mantenha a seleção da regra _lb_ 
![Autoscale load balancing](autoscale-loadbalancing.png)
5. Em __ScaleUp policies__, ajuste o __Duration__ e __Quite time__ para 30 segundos, para encurtar o tempo de resposta no nosso teste de carga. Em __Counter__ escolha _VM CPU - average percentage_, __Operator__: _Greater than_, __Threshold__: _70_ e pressione __Add condition__
![ScaleUp](scaleup.png)
5. Em __ScaleDown policies__, ajuste o __Duration__ e __Quite time__ para 30 segundos, para encurtar o tempo de resposta no nosso teste de carga. Em __Counter__ escolhe _VM CPU - average percentage_, __Operator__: _Less than_, __Threshold__: _20_ e pressione __Add condition__
![ScaleUp](scaledown.png)
6. Finalmente, nos detalhes, coloque o nome de sua escolha, __Expunge VM grace period__ de 30 segundos, para que as VMs sejam rapidamente apagadas após desativadas; mínimo e máximo de 1 a 4 membros; __Polling interval__ de 30 segundos também. Conclua clicando em __Create__.
![Autoscale Details](autoscale-details.png)
7. Aguarde a criação do _Autoscale Group_. Note que é necessário recriar as regras de firewall para o IP após ter sido remapeado para nova instância:
![Firewall template](firewall-template.png)

## Teste da aplicação

Você pode verificar o grupo criado no menu __Compute__, __AutoScale VM Groups__. E, também, em __Compute__, __Instances__, pode verificar que já foi criada uma instância com nome do tipo _autoScaleVm-grupo-autoscale-1-apknvq_.

Pode também testar as páginas, usando o endereço IP público associado ao load balancer criado.

```
http://xIP_WEBx/info.php
```
```
http://xIP_WEBx/todo.php
```
```
http://xIP_WEBx/cpu.php
```

## Teste de carga

Instale o gerador de carga "hey" com `sudo apt install hey` ou `brew install hey` e digite, substituindo pelo IP alocado:

```bash
hey -z 3m http://xIP_WEBx/cpu.php
```

### Acompanhamento do AutoScale

Durante o andamento do teste acompanhe o comportamento do AutoScale em __Compute__, __AutoScaleVM Groups__ e clicando em __Refresh__ à medida em que o teste ocorre. 
![Scaling](scaling.png)

Também é interessante acompanhar em __Compute__, __Instances__ como as VMs vão sendo paradas e apagadas depois que o teste termina.
![Stopping](stopping.png)

E, também, ao clicar sobre a instância, é possível ver o comportamento em __Metrics__:
![Metrics](metrics.png)
