# 🛡️ Análise de Vulnerabilidades em Laboratório

### Um Pentest controlado na Metasploitable 2

<p align="center">
  <img src="https://img.shields.io/badge/Kali_Linux-557C94?style=for-the-badge&logo=kali-linux&logoColor=white" alt="Kali Linux"/>
  <img src="https://img.shields.io/badge/VirtualBox-2D3748?style=for-the-badge&logo=virtualbox&logoColor=white" alt="VirtualBox"/>
  <img src="https://img.shields.io/badge/Nmap-000000?style=for-the-badge&logo=nmap&logoColor=white" alt="Nmap"/>
  <img src="https://img.shields.io/badge/Hydra-003366?style=for-the-badge&logo=hydra&logoColor=white" alt="Hydra"/>
  <img src="https://img.shields.io/badge/SQLMap-E34F26?style=for-the-badge&logo=sqlite&logoColor=white" alt="SQLMap"/>
</p>

## 📝 Sumário Executivo

Este projeto documenta uma análise de segurança ofensiva em um laboratório isolado. Usando o **Kali Linux** contra a máquina **Metasploitable 2**, detalho um processo completo de pentest: reconhecimento, enumeração de serviços, ataques de força bruta, exploração de falhas web e escalonamento de privilégios. O objetivo é demonstrar uma metodologia estruturada, identificar vulnerabilidades críticas e propor soluções, servindo como uma peça de portfólio técnico.

## 🚀 Índice

1.  [🎯 Overview do Projeto](#overview)
2.  [💻 Configuração do Ambiente](#ambiente)
3.  [🔎 Fase 1: Reconhecimento e Enumeração](#fase-1)
4.  [🔑 Fase 2: Ataques de Força Bruta](#fase-2)
5.  [💥 Fase 3: Exploração de Falhas na Web](#fase-3)
6.  [📈 Fase 4: Pós-Exploração e Análise de Hashes](#fase-4)
7.  [📊 Relatório de Ameaças e Recomendações](#relatorio)
8.  [💡 Conclusão e Aprendizados](#conclusao)

## <a id="overview"></a>🎯 Overview do Projeto

**Objetivos:**
* **Demonstrar Proficiência:** Usar ferramentas padrão da indústria (`Nmap`, `Hydra`, `Medusa`, `SQLMap`, `John The Ripper`) em um cenário prático.
* **Aplicar Metodologia:** Seguir as etapas de um pentest, do reconhecimento à pós-exploração.
* **Documentar Resultados:** Criar um relatório técnico claro e profissional para portfólio.
* **Recomendar Defesas:** Propor soluções realistas para cada vulnerabilidade encontrada.

**Ferramentas Utilizadas:**
* `nmap`: Mapeamento de rede e descoberta de serviços.
* `medusa`: Brute-force de senhas rápido e paralelo.
* `hydra`: Ferramenta de brute-force com suporte a múltiplos protocolos.
* `sqlmap`: Detecção e exploração automatizada de SQL Injection.
* `john`: John The Ripper para quebra de senhas offline.
* `VirtualBox`: Gerenciamento das máquinas virtuais.

## <a id="ambiente"></a>💻 Configuração do Ambiente

O laboratório foi montado em um ambiente isolado para garantir a segurança da rede local.

* **Virtualizador:** VirtualBox 7.1.12
* **VM Atacante:** Kali Linux | IP: `192.168.56.102`
* **VM Alvo:** Metasploitable 2 | IP: `192.168.56.101`
* **Rede:** Host-Only (`vboxnet0`), isolada de tráfego externo.

> **Boa Prática:** Antes de iniciar, um **snapshot** da VM Metasploitable 2 foi criado. Isso permite reverter a máquina a um estado limpo a qualquer momento, garantindo que os testes sejam consistentes e reproduzíveis.

## <a id="fase-1"></a>🔎 Fase 1: Reconhecimento e Enumeração

A fase inicial consiste em coletar o máximo de informações sobre o alvo. Para isso, usei o `Nmap` para identificar hosts, portas abertas, serviços e versões.

#### Nmap Scan
Realizei uma varredura TCP completa para mapear a superfície de ataque do alvo.

**Comando Executado:**
```bash
sudo nmap -p- -sV -sC -A -T4 -oN nmap_results.txt 192.168.56.101
```

### Resultados do Scan:

```
Nmap scan report for 192.168.56.101
Host is up (0.0015s latency).
Not shown: 65505 closed tcp ports (reset)
PORT      STATE SERVICE     VERSION
21/tcp    open  ftp         vsftpd 2.3.4
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 192.168.56.102
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp    open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
23/tcp    open  telnet      Linux telnetd
25/tcp    open  smtp        Postfix smtpd
|_smtp-commands: metasploitable.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_RC2_128_CBC_WITH_MD5
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_RC4_128_WITH_MD5
|     SSL2_DES_64_CBC_WITH_MD5
|_    SSL2_DES_192_EDE3_CBC_WITH_MD5
| ssl-cert: Subject: commonName=ubuntu804-base.localdomain/organizationName=OCOSA/stateOrProvinceName=There is no such thing outside US/countryName=XX
| Not valid before: 2010-03-17T14:07:45
|_Not valid after:  2010-04-16T14:07:45
|_ssl-date: 2025-10-17T17:38:32+00:00; -1s from scanner time.
53/tcp    open  domain      ISC BIND 9.4.2
| dns-nsid: 
|_  bind.version: 9.4.2
80/tcp    open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
|_http-server-header: Apache/2.2.8 (Ubuntu) DAV/2
|_http-title: Metasploitable2 - Linux
111/tcp   open  rpcbind     2 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/udp   nfs
|   100005  1,2,3      35168/tcp   mountd
|   100005  1,2,3      50370/udp   mountd
|   100021  1,3,4      35421/udp   nlockmgr
|   100021  1,3,4      55243/tcp   nlockmgr
|   100024  1          48648/tcp   status
|_  100024  1          49096/udp   status
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp   open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
512/tcp   open  exec        netkit-rsh rexecd
513/tcp   open  login
514/tcp   open  shell       Netkit rshd
1099/tcp  open  java-rmi    GNU Classpath grmiregistry
1524/tcp  open  bindshell   Metasploitable root shell
2049/tcp  open  nfs         2-4 (RPC #100003)
2121/tcp  open  ftp         ProFTPD 1.3.1
3306/tcp  open  mysql       MySQL 5.0.51a-3ubuntu5
| mysql-info: 
|   Protocol: 10
|   Version: 5.0.51a-3ubuntu5
|   Thread ID: 8
|   Capabilities flags: 43564
|   Some Capabilities: Support41Auth, SwitchToSSLAfterHandshake, ConnectWithDatabase, LongColumnFlag, Speaks41ProtocolNew, SupportsCompression, SupportsTransactions
|   Status: Autocommit
|_  Salt: P=!-[flkTE;6=o9l8nFN
3632/tcp  open  distccd     distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
5432/tcp  open  postgresql  PostgreSQL DB 8.3.0 - 8.3.7
|_ssl-date: 2025-10-17T17:38:32+00:00; -1s from scanner time.
| ssl-cert: Subject: commonName=ubuntu804-base.localdomain/organizationName=OCOSA/stateOrProvinceName=There is no such thing outside US/countryName=XX
| Not valid before: 2010-03-17T14:07:45
|_Not valid after:  2010-04-16T14:07:45
5900/tcp  open  vnc         VNC (protocol 3.3)
| vnc-info: 
|   Protocol version: 3.3
|   Security types: 
|_    VNC Authentication (2)
6000/tcp  open  X11         (access denied)
6667/tcp  open  irc         UnrealIRCd
6697/tcp  open  irc         UnrealIRCd
8009/tcp  open  ajp13       Apache Jserv (Protocol v1.3)
|_ajp-methods: Failed to get a valid response for the OPTION request
8180/tcp  open  http        Apache Tomcat/Coyote JSP engine 1.1
|_http-title: Apache Tomcat/5.5
|_http-favicon: Apache Tomcat
|_http-server-header: Apache-Coyote/1.1
8787/tcp  open  drb         Ruby DRb RMI (Ruby 1.8; path /usr/lib/ruby/1.8/drb)
35168/tcp open  mountd      1-3 (RPC #100005)
48648/tcp open  status      1 (RPC #100024)
49376/tcp open  java-rmi    GNU Classpath grmiregistry
55243/tcp open  nlockmgr    1-4 (RPC #100021)
MAC Address: 08:00:27:9C:2B:50 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6
OS details: Linux 2.6.9 - 2.6.33
Network Distance: 1 hop
Service Info: Hosts:  metasploitable.localdomain, irc.Metasploitable.LAN; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: metasploitable
|   NetBIOS computer name: 
|   Domain name: localdomain
|   FQDN: metasploitable.localdomain
|_  System time: 2025-10-17T13:38:24-04:00
|_nbstat: NetBIOS name: METASPLOITABLE, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
|_smb2-time: Protocol negotiation failed (SMB2)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: 59m59s, deviation: 2h00m00s, median: -1s

TRACEROUTE
HOP RTT     ADDRESS
1   1.49 ms 192.168.56.101

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 167.03 seconds
```

### Análise dos Resultados

A varredura revelou uma superfície de ataque gigante. As principais conclusões foram:

- **Exposição Crítica:** Um **bindshell na porta 1524** oferece acesso root direto e sem autenticação. Isso, por si só, já compromete totalmente o sistema.
- **Software Desatualizado:** Vários serviços rodam versões com vulnerabilidades conhecidas (ex: ```vsftpd 2.3.4``` com backdoor e ```UnrealIRCd``` com falhas de RCE).
- **Protocolos Inseguros:** Protocolos de texto plano como **FTP (21)** e **Telnet (23)** estão ativos, arriscando o vazamento de credenciais.
- **Múltiplos Vetores:** O alvo expõe serviços web (**80**, **8180**), bancos de dados (**MySQL 3306**, **PostgreSQL 5432**) e compartilhamento de arquivos (**SMB 445**), todos potenciais alvos.

Apesar do acesso direto pelo bindshell, decidi focar nos serviços FTP, SSH, HTTP e SMB para demonstrar uma metodologia de ataque mais detalhada.

### <a id="fase-2"></a>🔑 Fase 2: Ataques de Força Bruta

Com os serviços mapeados, o foco agora é descobrir credenciais fracas com ataques de força bruta.

### Serviço FTP com Medusa
- Alvo: vsftpd 2.3.4 na porta 21.
- Metodologia: Usei `Medusa` com uma wordlist simples para realizar um ataque de dicionário.

**Comandos Executados:**
```
echo "msfadmin" > users.txt
```
```
echo "user" >> users.txt
```
```
echo "root" >> users.txt
```
```
echo "password" > passwords.txt
```
```
echo "123456" >> passwords.txt
```
```
echo "msfadmin" >> passwords.txt
```
```
medusa -h 192.168.56.101 -U users.txt -P passwords.txt -M ftp
```

### Resultado: Sucesso! 
O Medusa encontrou as credenciais ``msfadmin:msfadmin``, que foram validadas com um login manual.

_Medusa encontrando as credenciais do serviço FTP._

![Sucesso](images/SUCESSMEDUSA.png)

### Serviço SSH com Hydra

**Alvo:** `OpenSSH 4.7p1` na porta 22.

**Metodologia e Desafios:** A primeira tentativa com `Hydra` falhou devido a um erro de handshake criptográfico (`kex error`). Isso ocorre porque o Kali, por padrão, desabilitou os algoritmos antigos e inseguros que o servidor SSH do Metasploitable 2 utiliza. Para resolver, tentei modificar a configuração SSH do cliente para permitir cifras mais fracas, mas a incompatibilidade persistiu. Em um cenário real, este vetor de ataque seria descartado, mas para fins de estudo, a solução foi contornada.

_Hydra descobrindo as credenciais SSH após ajustes de criptografia._

![Sucess](images/hydrasucess.png)

_Login manual no FTP confirmando o acesso._

![Sucesso](images/ftpsucess.png)

### Formulário Web (DVWA) com Hydra

**Alvo:** Formulário de login do DVWA na porta 80.

**Metodologia:** Usei o módulo `http-post-form` do `Hydra`. Analisei a requisição POST para identificar os parâmetros (`username`, `password`) e a mensagem de erro (`Login failed`).

**Comando Executado:**
```
hydra -L web_user.txt -P passwords.txt 192.168.56.101 http-post-form "/dvwa/login.php:username=^USER^&password=^PASS^&Login=Login:F=Login failed" -V
```

**Resultado:** O ataque encontrou a senha password para o usuário ```admin```. O acesso foi validado via navegador.

_Saída do Hydra mostrando a senha correta para o usuário ‘admin’._

![Sucess](images/dvwasucess.png)

_Login bem-sucedido na página do DVWA, confirmando as credenciais._

![Sucess](images/dvwaconnect.png)

## <a id="fase-3"></a>💥 Fase 3: Exploração de Falhas na Web

Nesta fase, o foco muda de autenticação para vulnerabilidades na aplicação, especificamente SQL Injection.

### SQL Injection com SQLMap

**Alvo:** Página "SQL Injection" do DVWA.

**Metodologia e Análise:** O objetivo era usar `sqlmap` para explorar a falha. No entanto, a ferramenta falhou repetidamente, mesmo com os cookies de sessão corretos.

**1. Falha Inicial:** `sqlmap` não conseguiu detectar a vulnerabilidade.

**Validação Manual:** Para confirmar que a falha existia, injetei manualmente a carga clássica `'1' OR '1'='1'`. O teste **funcionou**, e a aplicação retornou todos os usuários do banco de dados. Isso provou que a vulnerabilidade era real e o problema estava na interação da ferramenta com o alvo.

**Conclusão Profissional:** Mesmo com configurações agressivas, o `sqlmap` continuou falhando. A conclusão é que esta instância específica do `DVWA` tem um comportamento que impede a exploração por ferramentas automatizadas padrão. Em um pentest real, a vulnerabilidade seria reportada como confirmada manualmente, e o tempo seria alocado para outros vetores. Este processo destaca a importância de validar falhas manualmente e não depender cegamente de ferramentas.

_A injeção manual confirmou o SQLi quando as ferramentas automatizadas falharam._

![Success](images/manualsucess.png)

## <a id="fase-4"></a>📈 Fase 4: Pós-Exploração e Análise de Hashes
Após a exploração, o próximo passo é utilizar os dados obtidos, como os hashes de senha do banco de dados.

### Quebra de Senhas com John The Ripper

**Alvo:** Hashes MD5 e SHA256 extraídos da tabela `users`.

**Metodologia:** Usei `John The Ripper` para um ataque de quebra de senhas offline. Como o JTR não detectou o formato dos hashes corretamente, separei os hashes MD5 e SHA256 em arquivos diferentes e especifiquei o formato manualmente com a flag `--format`.

**Comandos Executados:**
#### Separar hashes por tipo
```
echo "5f4dcc3b5aa765d61d8327deb882cf99" > hashes_md5.txt
```
```
echo "8d969eef6ecad3c29a3a629280e686cf0c3f5d5a86aff3ca12020c923adc6c92" > hashes_sha256.txt
```

#### Executar o John com o formato correto
```
john --format=raw-md5 --wordlist=passwords.txt hashes_md5.txt
```
```
john --format=raw-sha256 --wordlist=passwords.txt hashes_sha256.txt
```

#### Mostrar senhas quebradas
```
john --show --format=raw-md5 hashes_md5.txt
```
```
john --show --format=raw-sha256 hashes_sha256.txt
```

**Resultado:** John The Ripper quebrou os hashes em segundos, revelando as senhas em texto plano. Isso mostra a fragilidade de usar algoritmos de hash rápidos e sem "salt", como o MD5.

_JTR quebrando os hashes MD5 e SHA256 após o formato ser definido manualmente._

![Success](images/johnmd5.png) ![Success](images/johnsha256.png)

## <a id="relatorio"></a>📊 Relatório de Ameaças e Recomendações
Aqui estão as vulnerabilidades críticas encontradas, priorizadas por impacto.

| Vulnerabilidade | Risco | Descrição e Impacto | Recomendações de Mitigação |
| :--- | :--- | :--- | :--- |
| **Credenciais Fracas e Reutilizadas** | <span style="color:red">**Crítico**</span> | Serviços como FTP, SSH e SMB usavam senhas fáceis de adivinhar. A mesma credencial (`msfadmin`) foi reutilizada, permitindo que uma única falha comprometesse múltiplos sistemas. | <ul><li>**Implementar Política de Senhas Fortes:** Exigir comprimento mínimo (12+), complexidade e histórico.</li><li>**Adotar Bloqueio de Contas:** Bloquear contas após 5 tentativas de login falhas.</li><li>**Habilitar MFA:** Exigir um segundo fator de autenticação para todos os serviços críticos.</li></ul> |
| **Armazenamento Inseguro de Senhas** | <span style="color:red">**Crítico**</span> | A aplicação web armazena senhas como hashes MD5 sem "salt". Este algoritmo é trivial de quebrar em ataques offline. | <ul><li>**Migrar para Hashing Moderno:** Refazer os hashes de todas as senhas usando algoritmos fortes como **Argon2**, **bcrypt** ou **scrypt**.</li></ul> |
| **SQL Injection (SQLi)** | <span style="color:red">**Crítico**</span> | A aplicação web era vulnerável a SQLi, permitindo que um atacante extraísse todo o banco de dados de usuários. | <ul><li>**Usar Queries Parametrizadas:** Esta é a defesa mais eficaz contra SQLi.</li><li>**Validar e Sanitizar Inputs:** Tratar toda entrada do usuário no lado do servidor.</li></ul> |
| **Serviços Desatualizados** | <span style="color:orange">**Alto**</span> | Serviços como `vsftpd 2.3.4` e `UnrealIRCd` possuem vulnerabilidades públicas que levam à execução remota de código. | <ul><li>**Manter um Programa de Patch Management:** Aplicar patches de segurança regularmente.</li><li>**Reduzir a Superfície de Ataque:** Desabilitar todos os serviços não essenciais.</li></ul> |
| **Protocolos Inseguros** | <span style="color:orange">**Alto**</span> | FTP e Telnet transmitem dados e credenciais em texto claro, facilitando a interceptação de dados na rede. | <ul><li>**Desativar Protocolos em Texto Claro:** Desabilitar Telnet e FTP padrão.</li><li>**Forçar Criptografia:** Substituí-los por **SSH**, **SFTP** ou **FTPS**.</li></ul> |

## <a id="conclusao"></a>💡 Conclusão e Aprendizados
Este projeto demonstrou um pentest completo e multifacetado. Seguindo uma metodologia estruturada, várias vulnerabilidades críticas foram identificadas e exploradas.

O maior aprendizado foi a importância de solucionar problemas de forma metódica. Quase todas as ferramentas (```Hydra```, ```sqlmap```, ```John The Ripper```) falharam inicialmente por problemas do mundo real, como incompatibilidades de criptografia ou detecção incorreta de hashes. Ao diagnosticar a causa raiz e adaptar a abordagem, foi possível superar os obstáculos.

Este exercício prova que um pentest eficaz não é sobre rodar scripts, mas sim sobre entender os protocolos, ter uma abordagem analítica e persistir na resolução de problemas.
