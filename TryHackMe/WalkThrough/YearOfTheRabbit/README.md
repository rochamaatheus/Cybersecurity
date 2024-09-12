# YearOfTheRabbit Walkthrough

## Resumo

Este é um walkthrough completo da sala **YearOfTheRabbit** no TryHackMe. Ele cobre as técnicas de enumeração, exploração e elevação de privilégios usadas para completar a sala e obter as flags de usuário e root.

- Nota: O IP da sala é único, seu IP pode ser diferente do meu.

## Passos e Técnicas

### 1. Varredura com Nmap

Primeiro, realizamos uma varredura básica com o **Nmap** para identificar portas abertas e serviços em execução na máquina alvo:

```bash
nmap 10.10.11.115
```

**Resultados**:

- **FTP** (21) - Aberta
- **SSH** (22) - Aberta
- **HTTP** (80) - Aberta

Isso nos dá uma lista de serviços para explorar mais adiante.

### 2. Enumeração de Diretórios com Gobuster

Usamos o **Gobuster** para enumerar diretórios ocultos no servidor web rodando na porta 80:

```bash
gobuster dir -u http://10.10.11.115/ -w /usr/share/dirb/wordlists/common.txt
```

**Resultados**:

- /assets - Diretório aberto onde existia um vídeo e uma folha de estilo (CSS)

**Achados**:

- Foi descoberto o caminho `/sup3r_s3cr3t_fl4g.php`, que contém dicas valiosas para a exploração.

### 3. Uso do BurpSuite para Interceptar Tráfego HTTP

Em seguida, utilizamos o **BurpSuite** para interceptar o tráfego e revelar diretórios ocultos adicionais. Através deste processo, encontramos um link para um arquivo de imagem (`Hot_Babe.png`) que contém informações úteis.

### 4. Analisando a Imagem com Strings

Analisamos o arquivo **Hot_Babe.png** utilizando o comando `strings` para buscar por informações ocultas. Isso revela credenciais de FTP.

```bash
strings Desktop/Hot_Babe.png
```

**Credenciais Encontradas**:

- Nome de usuário: `ftpuser`
- Várias senhas possíveis.

### 5. Força Bruta no FTP com Hydra

Utilizamos o **Hydra** para realizar um ataque de força bruta no serviço FTP usando as credenciais encontradas:

```bash
hydra -l ftpuser -P Desktop/passwords ftp://10.10.11.115
```

- Nota: Aqui criei um arquivo txt que continha as possíveis senhas (passwords)

**Resultado**: Encontramos a senha correta: `5iez1wGXKfPKQ`.

### 6. Conectando via FTP

Com as credenciais de FTP, nos conectamos ao servidor e baixamos o arquivo **Eli's_Creds.txt**.

```bash
ftp 10.10.11.115
get Eli's_Creds.txt
```

### 7. Decodificando Brainfuck para Encontrar Credenciais de SSH

O conteúdo do arquivo **Eli's_Creds.txt** estava codificado em **Brainfuck**. Após decodificar, obtivemos as credenciais de SSH para o usuário `eli`:

**Decodificado**:

- Usuário: `eli`
- Senha: `DSpDiM1wAEwid`

### 8. Acesso SSH como o Usuário Eli

Fazemos login via SSH com as credenciais decodificadas:

```bash
ssh eli@10.10.11.115
```

### 9. Encontrando Diretórios Ocultos para Gwendoline

Ao acessar a máquina como **eli**, encontramos uma mensagem sugerindo a existência de um "esconderijo s3cr3t". Usamos o comando `find` para localizar arquivos contendo o termo `s3cr3t`:

```bash
find / -name *s3cr3t* 2>/dev/null
```

### 10. Trocando para o Usuário Gwendoline

Encontramos um arquivo que revela a senha de **Gwendoline** e utilizamos o comando `su` para trocar para a conta dela:

```bash
su gwendoline
```

### 11. Escalada de Privilégios para Root

Usando o comando `sudo -l`, descobrimos que Gwendoline pode executar o editor **vi** com privilégios elevados. Exploramos isso para abrir um shell root:

```bash
sudo -u#-1 /usr/bin/vi /home/gwendoline/user.txt
:!bash
```

### 12. Obtendo a Flag de Root

Finalmente, navegamos até o diretório `/root` para obter a **flag de root**:

```bash
cat /root/root.txt
```

## Conclusão

Seguindo esses passos, conseguimos comprometer a máquina **YearOfTheRabbit**, elevar os privilégios a root e obter as flags de usuário e root.
