# Sobre a Instalação do Oracle Linux em uma máquina virtual

Para a instalação do Oracle linux em uma máquina virtual, 
foram usados os softwares a seguir:
1. Imagem de instalação do [Oracle Linux](https://yum.oracle.com/oracle-linux-isos.html) (ISO)
2. [Oracle VM VirtualBox](https://www.virtualbox.org/)

<details>
  <summary>Detalhes de instalação dos softwares</summary>
  
  ### Oracle Linux ISO
  1. As imagems (ISOs) do `Oracle Linux`, podem ser encontradas em:

     * https://yum.oracle.com/oracle-linux-isos.html

  ### Oracle VM VirtualBox
  1. O Software de virtualização `Oracle VM VirtualBox`, pode ser encontrado em:

     * https://www.virtualbox.org/wiki/Downloads

    OBS: A Instalação do Oracle VM VirtualBox pode ser diferente dependendo do sistema operacional utilizado, atenção para as instruções no site do virtual box!
</details>

</br>

Após a instalação do `Oracle VM VirtualBox` e com a ISO do `Oracle Linux` baixada, 

---

## Configuração da Máquina Virtual
No software `Oracle VM VirtualBox`, pode-se criar uma nova máquina virtual e alocar, no mínimo: 
* 2Gb de memória ram
* 12Gb de Armazenamento em VDI (Virtual Disk Image) dinamicamente alocado.

E, após a criação da VM em questão, selecionando ela no VM VirtualBox e em 'Configurações' -> Rede,
selecione o adaptador 1 para modo BRIDGE, selecione sua placa de rede.

---

## Configuração e instalação do Oracle Linux na máquina virtual

E então, inicie a VM em questão no VM VirtualBox, selecione a imagem de instalação do `Oracle Linux` (ISO).

Após isso, instalaremos o `Oracle Linux` apartir da mídia de instalação com o auxilio da interface, as configurações abaixo podem ser feitas em qualquer ordem:

* Em 'Software Selection', selecione a opção de 'Minimal Install' para uma instalação minimalista e sem Interface Gráfica (GUI).

* Em 'Network & Host Name' ative a interface de rede (que por padrão vem desativada).

* Em 'Keyboard', selecione a configuração de teclado que preferir (Exemplo: Portuguese Brasil)

* Em 'Installation Destination', selecione a opção de 'Storage Configuration' como 'automatic'.

* Em 'User Settings'
Vá na opção 'Root Password' e coloque uma senha para o usuário root.

* Em 'User Settings'
Vá na opção 'User Creation' e crie os usuários: 
  * 'comp_admin' com uma senha e com privilégios de administrador.
  * 'comp_user' com uma senha 


Após isso, pode clicar na opção de 'Begin installation' para começar a instalação do oracle linux com as configurações acimas.

Então, ao entrar na máquina, logar com um usuário administrador
e confirmar que há acesso a internet,
procure por atualizações disponíveis e 
faça a instalação delas se houverem,
por meio dos comandos:

```
sudo yum check-update
sudo yum update
```

E, pronto, o `Oracle Linux` está instalado, configurado e pronto para uso !

---

## Configuração da Relação de Confiança entre VMs
Para isso, usaremos duas Máquinas Virtuais, ambas com a instalação acima do Oracle Linux:
* Máquina Cliente
* Máquina Servidor

E, então,
Instalaremos o `policycoreutils-python-utils` em ambas as máquinas, com o comando:

```
sudo yum install policycoreutils-python-utils
```

e então, antes de começarmos a fazer a relaçao de confiança, 
faremos algumas configurações de segurança,

Primeiro vamos acessar o arquivo de configuração do deamon ssh
```
sudo vi /etc/ssh/sshd_config
```
Mudaremos a porta padrão do ssh, escolhendo uma porta diferente da padrão 22, por exemplo, a porta 22005.
```
port 22005 
```
Vamos desabilitar a opção
que permite fazer login como usuário root
```
PermitRootLogin no
```
Agora, para cliente e servidores
se comunicarem sem problemas através da 
nova porta, vamos abrir a porta no firewall
e no SElinux
da máquina de servidor.

```
sudo firewall-cmd --add-port=22005/tcp --permanent
semanage port -a -t ssh_port_t -p tcp 22005
```
---

E então, para criarmos a relação de confiança, na **Máquina Cliente**
vamos nos logar com o usuário desejado, nesse caso, 'comp_user', e então, usaremos o comando
"ssh-keygen" para gerarmos as chaves e credenciais. ( Por padrão, ele guarda as chaves no diretório oculto '.ssh/' que fica na pasta $HOME do usuário atualmente logado na secção)

```
ssh-keygen
```

E então, passaremos as informações para a 
a máquina de servidor com o comando ssh-copy-id, no formato:
```
ssh-copy-id -i <local_da_chave_pública> <usuário_servidor@ip_servidor>
```

Exemplo ('comp_user' sendo um usuário do servidor e '192.168.1.30' como o ip do servidor):
```
ssh-copy-id -i ~/.ssh/id_rsa.pub comp_user@192.168.1.30
```

e após entrarmos com a senha do usuário 'comp_user' do servidor, deve aparecer a mensagem dizendo que
a chave foi adicionada ao servidor.

E agora, podemos logar sem precisar de senha,
pois efetuamos a relação de confiança

Para fazermos a conexão com a máquina servidor, podemos usar o comando:
```
ssh <usuário_servidor@ip_servidor> -p <porta>
```
Exemplo:
```
ssh comp_user@192.168.100.38 -p 22005
```

---
