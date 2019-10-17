# Ansible
Vamos imaginar que temos 1 servidor para realizar a configuração dele, não teriamos problemas em configurar ele e deixar tudo funcionando perfeitamente. Mas e se tivessemos com 3 ? Um pouco mais trabalhoso, mesmo assim não teriamos problemas e se tivessemos 100 ou 1000 ? Usando o **Ansible** conseguiriamos realizar a mesma configuração em todos os servidores e se tivermos algum erro conseguiriamos ter um feedback de erro.

## Mas afinal o que é **Ansible**?  
O Ansible é um sistema de automação de configurações feito em Python, que nos ajuda e nos permite escrever descrever procedimentos em arquivos no formato **YAML** que são reproduzidos usando SSH em maquinas remotas. Dessa forma só precisamos ter acesso ao SSH do servidor que queremos instalar e assim realizar os comandos de forma remota.

## Outras ferramentas
Existem outras ferramentas nessa categoria ...

- Chef
- Puppet

## Instalação
Vamos instalar o **Ansible** em nossa maquina local, ele não necessita de nenhum client/agent/biblioteca que deva ser instalado na maquina virtual. O **Ansible** é feito em python e funciona em diversas plataformas.

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
- atualizar pacotes
- instalar nginx


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

### Criando e adicionando chave publica

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
