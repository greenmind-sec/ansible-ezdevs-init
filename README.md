# Ansible
Vamos imaginar que temos 1 servidor para realizar a configuração dele, não teriamos problemas em configurar ele e deixar tudo funcionando perfeitamente. Mas e se tivessemos com 3 ? Um pouco mais trabalhoso, mesmo assim não teriamos problemas e se tivessemos 100 ou 1000 ? Usando o **Ansible** conseguiriamos realizar a mesma configuração em todos os servidores e se tivermos algum erro conseguiriamos ter um feedback de erro.

## Documentação oficial
Nada melhor que a documentação oficial para tirar duvidas e novidades de novas versões.
```sh
https://docs.ansible.com/
```

## Mas afinal o que é **Ansible**?  
O Ansible é um sistema de automação de configurações feito em Python, que nos ajuda e nos permite escrever descrever procedimentos em arquivos no formato **YAML** que são reproduzidos usando SSH em maquinas remotas. Dessa forma só precisamos ter acesso ao SSH do servidor que queremos instalar e assim realizar os comandos de forma remota.

## Outras ferramentas
Existem outras ferramentas nessa categoria ...

- Chef
- Puppet

## Instalação
Vamos instalar o **Ansible** em nossa maquina local, ele não necessita de nenhum client/agent/biblioteca que deva ser instalado na maquina remota. O **Ansible** é feito em python e funciona em diversas plataformas.

### Como se comunica ?
O Ansible se comunica com as maquinas virtuais ou fisicas usando conexão SSH.

### Instalando MacOS X
Conseguimos instalar no MacOS via homebrew, só precisamos realizar o **brew install ansible**.
```sh
brew install ansible
```

### Instalando no Linux - Debian
Conseguimos instalar no Debian usando o **pip** e pode ser feito da seguinte forma.
```sh
sudo easy_install pip
sudo pip install ansible
```

### Instalando no Linux - Ubuntu
Conseguimos tambem instalar no Ubuntu usando **apt-get**.
```sh
sudo apt-get install software-properties-common
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get update
sudo apt-get install ansible
```

## Testar se está OK
Vamos realizar um teste e ver se ele está funcionando normalmente.
```sh
ansible-playbook -h
```

## A configuração do ansible e o formato yaml
No arquivo .yaml tem informações, como por exemplo
- hosts (Quais hosts vão ser usados)
- sudo (O comando vai usar sudo ?)
- user (Qual usuario vamos usar)
- tasks (Quais os passos que vão ser feitos)

### Criando primeiro exemplo (Atualizar e instalar nginx)
Quais hosts esse comando vai ser executado ?
- Nesse caso em todos
- Vamos usando o **sudo** ?
- Qual usuario vai ser usado?

Nesse exemplo vamos realizar dois passos e são eles
- Atualizar pacotes
- Instalar nginx

Vamos salvar esse arquivo com o nome **webserver.yml**.
```sh
- hosts: all
  sudo: True
  user: admin
  tasks:
    - name: "Atualiza pacotes"
      shell: sudo apt-get update

    - name: "Instala o nginx"
      shell: sudo apt-get -y install nginx
```
> Esse é o formato do arquivo **yaml**. Por padrão usamos dois espaço como **tab**.

Precisamos prestar atenção ao sinal **:** (dois pontos) após o atributo. Caso de algum erro o Ansible vai reclamar e assim podemos arrumar ele.

### Analisando
Vamos analisar o **webserver.yml**.

Analisando a primeira linha temos o **hosts**, ele é é o nivel mais alto do playbook. Com ele conseguimos definir qual a maquina ou grupo de maquinas a ação será feita.

Com o uso do valor **all**, mostra que pode ser aplicado a qualquer um definida no arquivo que contem os endereços das maquinas.

Se nós fossemos usar o playbook diretamente no terminal vamos precisar passar o arquivo com os endereços das maquinas.
```sh
[grupo1]
10.0.0.1
10.0.0.2
10.0.0.3

[grupo2]
10.0.0.4
10.0.0.5
```
> Se o valor do **hosts** fosse **grupo1**, somente os servidores do **grupo1** receberia as modificações.


### Executando o Ansible
Podemos executar o **Ansible** de forma manualmente e poderiamos usar o comando **ansible-playbook -i hosts_file** seguido no nosso playbook.
```sh
ansible-playbook -i hosts_file webserver.yml
```
> O comando **ansible-playbook -i hosts_file** é usado para que possamos passar um determinado grupo de hosts.

> E **hosts_file** é o nosso arquivo contendo os grupos de hosts.

### Entendendo melhor hosts
O nosso playbook só vai ser aplicado nas maquinas que tenham os IPS:
- 10.0.0.1
- 10.0.0.2
- 10.0.0.3.
> Caso tenhamos setado **grupo1**.

Se fosse **all** ele seria aplicado em todas as maquinas.

Dessa forma conseguimos configurar apenas um grupo determinado de servidores, assim caso tenha diversos servidores, maquinas virtuais e queira executar o playbook.

### Chave SSH
Tudo o que precisamos nos clientes é que o usuario tenha autenticação por chave SSH, por exemplo:
```sh
ssh usuario@maquina
```
> Ele deve logar sem pedir a senha. Precisamos modificar o usuario do arquivo do **Ansible** e execute o playbook como indicado anteriormente.

É muito importante que o usuario não precise de senha para logar, podemos resolver isso adicionando a public key no **authorized_keys**.

Alem disso devemos configurar para que não seja necessario o uso de senha no **sudo**.

### Retirando acesso via senha por SSH
Na Distribuição Debian e seus derivados podemos alterar o arquivo de configuração do nosso servidor e assim retirar o login por senha.
```sh
sudo nano /etc/ssh/sshd_config
```

Vamos procurar por **PasswordAuthentication yes**, pode estar comentado, vamos descomentar e mudar seu valor para **no**.

Segue o exemplo
```sh
PasswordAuthentication no
```
> Dessa forma não vamos mais nos autenticar com o uso de senha.

### Criando e adicionando chave publica
Vamos na nossa maquina host criar uma chave publica, vamos usar
```sh
ssh-keygen
```
> Por padrão ele cria o arquivo **id_rsa**.

Podemos apertar enter para deixar o nome padrão **id_rsa**, ou então dar um nome para essa chave.

Vamos até o diretorio do **.ssh**, se estiver usando o Debian ou algum derivado vai ser parecido com esse caminho.
```sh
/home/greenmind/.ssh
```
> **greenmind** é meu nome de usuario, caso o seu seja **carlos** seria algo como **/home/carlos/.ssh**.

Nossa chave publica vai ter o final **.pub**. Veja o exemplo da minha chave:
```sh
cat key_teste.pub
```

O resultado vai ser algo parecido com o resultado abaixo:
```sh
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC7h89rHd96PC/9M3qG59HoncTGopoB3qpUXWxltp1lcE54iCKAVZfx7C4eEJO/dFDWRiNm5XyMyCuOsYmEelnXLopSFSFTT$%Y#W@423546ygdwdw3ef44444TqWGKBocsmX8IQTdHuWcYOREaxTs9+WPdvPdc5uYqQ5ggVo7rP6K1VeJm/KbnCOhp7un/VWrKLYDXaRupaodaIKpFJnrA/4/DeS5fumI1lva/87HTLN greenmind@abase
```

Agora vamos até a maquina que queremos acessar, ir até o diretorio do SSH, procurar pelo arquivo **authorized_keys**, caso não tenha vamos criar.
```sh
sudo nano /home/greenmind/.ssh/authorized_keys
```
> **greenmind** é meu nome de usuario, caso o seu seja **carlos** seria algo como **/home/carlos/.ssh**.

Vamos adicionar a chave publica criada anteriormente nesse arquivo **authorized_keys** e dessa forma não precisamos mais de senha para logar no servidor.

### Testando acesso
Agora só realizar um login ao nosso servidor, se der tudo certo ele não vai pedir a senha, apenas para aceitar o uso da chave.
> Não se esqueça de testar a conexão antes, pois pode ter problemas ao executar e excutar a chave.

```sh
ssh user@meuservidor
```

### Conhecendo melhor os atributos
Vamos entender o
```sh
sudo: True
user: usuario
```
> Esses atributos indicam que o **Ansible** pode usar o **sudo** e o **usuario** é o user que vamos usar para o SSH.

Caso estejamos usando uma VPS, usariamos o usuario **default** da distribuição.
```sh
tasks:
  - name: "Atualiza pacotes"
    shell: sudo apt-get update
  - name: "Instala o nginx"
    shell: sudo apt-get -y install nginx
```

O **tasks** é uma lista de tarefas que o **Ansible** vai executar.
Ele é composta de **nome/ação**. Vamos criar uma **task** chamada **shell** que executa o comando em seguida diretamente no terminal.

### Primeiro exemplo completo
Vamos criar um arquivo chamado **hosts.txt**.
```sh
[test]
localhost
```

Agora vamos adicionar o **Atualiza.yaml**.
```sh
- hosts: all
  sudo: True
  user: greenmind
  tasks:
    - name: "Atualiza pacotes"
      shell: sudo apt-get update
```

Podemos fazer executar da seguinte forma
```sh
ansible-playbook -i hosts.txt atualiza.yaml
```

Podemos ver a saida da seguinte forma
```sh
[DEPRECATION WARNING]: Instead of sudo/sudo_user, use become/become_user and make sure become_method is 'sudo' (default). This feature will be removed in version 2.9. Deprecation warnings
can be disabled by setting deprecation_warnings=False in ansible.cfg.

PLAY [all] ***********************************************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************************************
ok: [localhost]

TASK [Atualiza pacotes] **********************************************************************************************************************************************************************
 [WARNING]: Consider using 'become', 'become_method', and 'become_user' rather than running sudo

changed: [localhost]

PLAY RECAP ***********************************************************************************************************************************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

**PLAY RECAP** é o resumo da aplicação do playbook.
Podemos ver que teve 2 ok.

## Conclusão
Vimos que com o uso do Ansible podemos aumentar a nossa produtividade, a mesma configuração realizada em um servidor vai ser feita em grupo especifico ou até todos os servidores que estejam no arquivos hosts. Com o uso de **infraestrutura como codigo** conseguimos escrever tudo o que nossa infraestrutura precisa e assim tendo a certeza que vai funcionar em todos os servidores. Desde pequenos processos de atualização, instalação de um simples pacote e até a instalação completa de uma infraestrutura de produção.

Qualquer coisa fico a disposição para ajudar, tirar duvidas e no que eu puder ajudar.

Obrigado e até a proxima.
