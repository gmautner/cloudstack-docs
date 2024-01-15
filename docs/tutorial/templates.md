# Templates e Userdata

Neste passo demonstraremos:

- Como criar __templates__, análogos às _imagens_ das nuvens públicas
- Usando __userdata__ para customizar instâncias a partir de templates

Utilizaremos os recursos criados no passo anterior, [DR e Snapshots](snapshots.md). Execute-o se ainda não o fez.

## Instalação da aplicação

!!! info
    O exemplo que segue é ilustrativo e a aplicação é muito simples. Mas a lógica serve para aplicações de qualquer natureza e complexidade, sejam monolitos, microsserviços, _back-ends_ etc.

Para ilustrar o funcinamento de _port forwarding_ entre via SSH no servidor _web_:

```bash
# Substitua o endereço IP abaixo pelo que foi configurado com port forwarding acima
ssh root@200.234.208.5 -p 22000
```

E instale o PHP para Apache:
```bash
apt install php libapache2-mod-php php-mysql
```

Para testar,
```bash
cat << 'EOF' > /var/www/html/info.php
<?php
phpinfo();
?>
EOF
```

E acesse `http://200.234.208.120/info.php` (aqui usamos o IP mapeado via _Static NAT_ para a instância _web_).

Agora criaremos nossa aplicação, que lista a tabela `todo_list`:

```bash
nano /var/www/html/todo.php
```

Copie o conteúdo, substituindo o endereço IP pelo do servidor _bd_, que pode ser lido no painel do CloudStack, e pela senha que utilizou para o banco:

```php
<?php
$user = "example_user";
$password = "<senha_bd>";
$database = "example_database";
$table = "todo_list";
$host = "10.1.1.120"; // Coloque o IP privado do servidor bd no CloudStack

try {
  $db = new PDO("mysql:host=$host;dbname=$database", $user, $password);
  echo "<h2>TODO</h2><ol>"; 
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}
?>
```

!!! tip "Dica"
    Note que usamos o IP privado do bd, pois o servidor web o acessa pela rede privada. Com isso, não precisamos expor o bd para a rede pública.

E acesse `http://200.234.208.120/todo.php` (use o IP mapeado via _Static NAT_ para a instância _web_).

### Página com CPU alta

Para testar o autoscaling mais adiante, criaremos uma página com alto consumo de CPU. Não se preocupe com o conteúdo.

!!! tip "Curiosidade"
    Eu pedi para o ChatGPT escrever uma página em PHP que consome muita CPU e gera resultados aleatórios. Ele respondeu com um método estatístico para calcular o valor de Pi. 

Na sessão SSH da instância _web_ execute:

```bash
nano /var/www/html/pi.php
```
E adicione o conteúdo:
```php
<?php
// Define the number of random points to be generated
$numPoints = mt_rand(1e6, 1e7);

$insideCircle = 0;

// Generate random points and check if they are inside the unit circle
for ($i = 0; $i < $numPoints; $i++) {
    $x = mt_rand(0, mt_getrandmax() - 1) / mt_getrandmax();
    $y = mt_rand(0, mt_getrandmax() - 1) / mt_getrandmax();

    if (sqrt($x * $x + $y * $y) <= 1) {
        $insideCircle++;
    }
}

// Estimate the value of Pi using the Monte Carlo method
$piEstimation = (4 * $insideCircle) / $numPoints;

echo "Estimated value of Pi: $piEstimation";
?>
```

Teste a página fazendo refresh no endereço `http://200.234.208.120/pi.php` algumas vezes e vendo o resultado mudar (use o IP mapeado via _Static NAT_ para a instância _web_).

## Userdata

No código acima, o endereço do _host_ do banco de dados e a senha estão hardcoded, impedindo que o servidor sirva de _template_ para deployment em outros ambientes ou que seja replicado para _load balancing_.

Então:

1. Edite o arquivo `/var/www/html/todo.php` para:
```php
<?php
include '/var/www/config.php'; // Linha acrescentada

$user = "example_user";
$password = DB_PASSWORD; // Valor hardcoded substituído por uma variável
$database = "example_database";
$table = "todo_list";
$host = DB_HOST; // Valor hardcoded substituído por uma variável

try {
  $db = new PDO("mysql:host=$host;dbname=$database", $user, $password);
  echo "<h2>TODO</h2><ol>"; 
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}
?>
```

2. Crie um novo arquivo fora da raiz do site:
```bash
nano /var/www/config.php
```
Insira o conteúdo (use o IP e senha do seu banco de dados):
```php
<?php
define('DB_HOST', '10.1.1.120'); // Coloque o IP privado do servidor bd no CloudStack
define('DB_PASSWORD', '<senha_bd>');
?>
```
E proteja as permissões:
```bash
chown www-data:www-data /var/www/config.php
chmod 600 /var/www/config.php
```
Acesse novamente `http://200.234.208.120/todo.php` para testar (use o IP mapeado via _Static NAT_ para a instância _web_).

3. Finalmente, apague o arquivo criado:
```bash
rm /var/www/config.php
cloud-init clean
```
Note que a aplicação quebrou. O segundo comando instrui o servidor a carregar o _Userdata_ no próximo boot.

4. No menu de navegação à esquerda clique em __Compute__, __Instances__ e clique na instância _web_
5. Clique no botão __Stop instance__ e confirme.
![Stop instance](stop.png)
6. Clique no botão __Edit instance__
![Edit instance](edit.png)
7. No campo __Userdata__, cole, substituindo o IP interno do servidor _bd_:
```yaml
#cloud-config

write_files:
  - path: /var/www/config.php
    content: |
      <?php
      // Use o IP privado do servidor bd no CloudStack
      define('DB_HOST', '10.1.1.120');
      define('DB_PASSWORD', '<senha_bd>');
      ?>
    owner: "www-data:www-data"
    permissions: '0600'
```
![Userdata](userdata.png)
8. Reinicie o servidor. Após aguardar alguns instantes:
    - Reinicie a sessão SSH e verifique que o arquivo `var/www/config.php` tem o conteúdo conforme acima
    - __Verifique que a página funciona normalmente__

!!! question "O que fizemos até agora?"
    Recapitulando:

    - Separamos as configurações num arquivo `config.php` à parte, fora da raiz do site
    - Usamos o mecanismo de _cloud-init_ para carregar as configurações a partir de uma definição na instância.
    - __O que falta__: agora transformaremos a instância num __template__ sem configurações. Este template poderá ser carregado com diferentes campos de _Userdata_ refletindo diferentes configurações, exemplo, _dev_, _staging_, _production_  

## Criação do template

1. Não queremos configurações no template, portanto execute via SSH na instância _web_:
```bash
rm /var/www/config.php
cloud-init clean --logs
```
O segundo comando limpa vestígios de carregamentos anteriores do _cloud-init_.

2. Pare a instância.

3. No menu de navegação à esquerda clique em __Compute__, __Instances__, clique na instância _web_ e em __Volumes__. Clique no link do volume (_ROOT-XXXX_)
![Volumes](volumes.png)
4. No canto superior direito, selecione __Create template from volume__
![Create template from volume](template-from-volume.png)
5. Preencha da seguinte forma e dê OK:
![To Do template](todo-template.png)

## Userdata com variáveis

No próximo passo, criaremos um __Userdata__ parametrizável, ou seja, onde o usuário pode escolher os valores `DB_HOST` e `DB_PASSWORD`

1. No menu de navegação à esquerda clique em __Compute__, __User Data__, __Register a userdata +__
2. No nome, colocar _tutorial_. Em __Userdata__ colar o código abaixo:
```yaml
## template: jinja
#cloud-config

write_files:
  - path: /var/www/config.php
    content: |
      <?php
      define('DB_HOST', '{{ ds.meta_data.db_host }}');
      define('DB_PASSWORD', '{{ ds.meta_data.db_password }}');
    owner: "www-data:www-data"
    permissions: '0600'
```
![Register userdata](register-userdata.png)
3. Antes de dar OK preencha o campo __Userdata parameters__ com `db_host, db_password`

!!! info
    Na primeira linha `## template: jinja` configuramos o _cloud-init_ para processar os valores que o CloudStack passa dentro das chaves `{{ }}`. Isto permite a parametrização do _Userdata_.

## Recriação da instância

Agora criaremos uma nova instância a partir do template:

1. No menu de navegação à esquerda clique em __Compute__, __Instances__
2. Clique no botão __Add instance +__
3. Em __Account__ coloque a sua conta.
4. Em __Templates__, escolha __My templates__ e escolha __To Do app__ 
![My templates](my-templates.png)
5. Em __Compute offering__ escolha __TBD__ (criar offers com CPU/memória fixas)
6. Em __Data disk__ mantenha __No thanks__
7. Em __Networks__ escolha a rede que criou, _minha-rede_
8. Em __SSH key pairs__ escolha a chave criada no passo anterior, por exemplo, _minha-chave_
![SSH key pairs](choose-keypair.png)
9. Em __Advanced mode__, habilite __Show advanced settings__
10. Em __Stored Userdata__, selecione __tutorial__ e preencha __db_host__: `10.1.1.120` (coloque o IP privado do servidor bd no CloudStack); __db_password__: `<senha_bd>`
![Stored Userdata](stored-userdata.png)
11. Em __name__ coloque _teste-template_ e clique __Launch instance__
12. Em __Compute__, __Instances__ verifique que a instância recém criada a partir do template está ligada e a anterior desligada
![Template running](template-running.png)
13. No menu à esquerda acesse __Networks__, __Guest networks__, _minha-rede_, __Public IP addresses__ e clique no endereço IP que fora mapeado via _Static NAT_ para o servidor _web_
![Select IP](select-ip.png)
14. Clique sobre o IP. Vamos desvincula-lo da instância _web_:
![Disable static NAT](disable-static-nat.png)
15. E agora vincule o mesmo IP à nova instância _teste-template_ seguindo o mesmo procedimento clicando em _Enable Static NAT_ e escolhendo-a como destino.
16. Note que é necessário recriar as regras de firewall para o IP após ter sido remapeado para nova instância:
![Firewall template](firewall-template.png)

Usando o endereço IP recém remapeado, acesse-o no browser em `http://200.234.208.120`. Deve aparecer a página padrão do Apache.

Acesse também as páginas, substituindo pelo endereço IP recém adquirido:

```
http://200.234.208.120/info.php
http://200.234.208.120/todo.php
http://200.234.208.120/pi.php
```

Se seguiu os passos até aqui, tudo deve funcionar, demonstrando que o servidor criado a partir do template possui toda a programação inserida previamente.

!!! tip "Lembrete"
    Certifique-se de ter remapeado via _Static NAT_ o endereço IP designado para a nova instância _teste-template_, criada a partir do template. Certifique-se de que a instância anterior _web_ esteja desligada para ter certeza de que está acessando a nova instância criada a partir do template.