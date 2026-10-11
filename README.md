# Atividade 1 – Documentação e Configuração: Acesso Remoto (SSH) e Firewall (UFW)

**Disciplina:** Gerência e Configuração de Serviços para Internet (GCSI)
**Aluno:** Francisco Barbosa da Silva Neto
**Ambiente:** Docker com imagem Ubuntu 24.04

---

## 1. Introdução

Esta documentação tem como objetivo detalhar o procedimento completo para a configuração de um ambiente isolado utilizando Docker, implementando um servidor de acesso remoto via **SSH** e regras de segurança de rede com o **Firewall UFW** em um container Ubuntu.

---

## 2. Ambiente Utilizado

| Item | Especificação |
| :--- | :--- |
| **Sistema Operacional do Container** | Ubuntu 26.04 LTS / 24.04 |
| **Ambiente de Execução** | Docker |
| **Serviço SSH** | OpenSSH Server |
| **Firewall** | UFW (Uncomplicated Firewall) |
| **Usuário Criado** | `gcsi` (com privilégios de sudo) |
| **Porta SSH no Container** | `22` |
| **Porta Mapeada no Host** | `2222` |
| **Nome do Container (Servidor)** | `ubuntu2604_no4` |
| **Nome do Container (Cliente)** | `ubuntu2604_no5` |

---

## 3. Criação do Ambiente Docker (Servidor)

Para que o UFW funcione corretamente dentro de um container Docker (permitindo o gerenciamento de regras de iptables/redes), criamos o container utilizando a flag `--privileged` e mapeando a porta SSH para o host:

```bash
docker run -d --name ubuntu2604_no4 --privileged -p 2222:22 ubuntu:26.04 bash -c "tail -f /dev/null"

Explicação dos parâmetros:

    -d: Executa o container em segundo plano (background).

    --name ubuntu2604_no4: Atribui um nome identificador amigável ao container.

    --privileged: Concede privilégios estendidos necessários para manipular regras de firewall e adaptadores de rede internos.

    -p 2222:22: Faz o redirecionamento de portas, ligando a porta 2222 da sua máquina física à porta 22 do container SSH.

    ubuntu:26.04: Define a imagem base utilizada.

    bash -c "tail -f /dev/null": Mantém o container ativo indefinidamente.

Para acessar o terminal interativo do container recém-criado como usuário root:

    docker exec -it ubuntu2604_no4 /bin/bash

4. Instalação dos Pacotes Necessários

Com o terminal aberto dentro do container, atualizamos os repositórios de pacotes e instalamos as ferramentas essenciais para o gerenciamento de rede, edição e segurança:

    apt update && apt install -y nano openssh-server ufw iptables iproute2

Finalidade de cada pacote:

    openssh-server: Permite que o container receba conexões remotas seguras via SSH.

    ufw: Interface simplificada para gerenciar as regras de firewall do sistema.

    nano: Editor de texto leve e prático para modificar arquivos de configuração.

    iptables / iproute2: Ferramentas essenciais de manipulação de pacotes de rede e inspeção de sockets (como o comando ss).

5. Criação e Configuração de Usuário

Por questões de segurança, é recomendável não utilizar o usuário root diretamente para conexões de rotina. Criaremos o usuário gcsi:

    useradd -m -s /bin/bash gcsi
    usermod -aG sudo gcsi
    passwd gcsi
(Defina uma senha segura para o usuário gcsi, por exemplo: 123456).

Defina também uma senha para o usuário root:

    passwd

6. Configuração do Servidor SSH

O arquivo de configuração principal do serviço SSH é o /etc/ssh/sshd_config. Vamos ajustá-lo para permitir o acesso adequado:

    nano /etc/ssh/sshd_config

Alterações essenciais no arquivo:

    1. Certifique-se de descomentar ou alterar a linha de permissão do root (caso necessário para testes):

        PermitRootLogin yes

    2. Garanta que a autenticação por senha vazia esteja desativada:

        PermitEmptyPasswords no
    Salve o arquivo (Ctrl + O, Enter) e saia (Ctrl + X).

Gerenciamento do Serviço SSH:
    Para iniciar, verificar o status ou reiniciar o serviço SSH dentro do container:

        service ssh start
        service ssh status

    Para verificar se o serviço está escutando na porta correta:

        ss -lntp

7. Configuração e Ativação do Firewall (UFW)

O UFW atua bloqueando tráfegos indesejados e permitindo apenas o que for explicitamente autorizado.

    7.1. Definir as Políticas Padrão

    Bloqueia todas as entradas por padrão e permite todas as saídas:

        ufw default deny incoming
        ufw default allow outgoing

    7.2. Liberar a Porta SSH

    Permite o tráfego TCP destinado à porta padrão do SSH:

        ufw allow 22/tcp

    7.3. Ativar o UFW

        ufw enable

    7.4. Verificar o Status das Regras

        ufw status verbose

8. Testes de Acesso Remoto

    8.1. Teste Local (Dentro do próprio container)

        ssh gcsi@localhost

    8.2. Teste a partir da Máquina Hospedeira (Host)

    Utilizando a porta mapeada pelo Docker (2222), acesse o container diretamente do terminal do seu Ubuntu físico:

        ssh -p 2222 gcsi@localhost

        O parâmetro -p 2222 indica ao cliente SSH que a conexão deve ser direcionada para a porta mapeada no host, que fará o redirecionamento seguro para a porta 22 interna do container.

9. Referências Oficiais

    Docker Documentation: Container networking & Port publishing

    Ubuntu Server Documentation: OpenSSH Server Guide e UFW Firewall

    OpenBSD Project: sshd_config(5) manual page


