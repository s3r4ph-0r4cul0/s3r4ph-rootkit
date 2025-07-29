`
# Análise Detalhada do Arquivo `azazel.c` do Rootkit Azazel

Este documento contém uma análise completa do código-fonte `util.c` do projeto Azazel. O foco é explicar o que cada parte do código faz, por que foi implementada dessa forma e referências para aprofundamento.

---

## 🧠 Visão Geral

Este arquivo contém funções utilitárias para o rootkit Azazel. Ele inclui rotinas como leitura de arquivos, checagem de permissões, leitura de entradas, e manipulação de processos — funções comuns para rootkits em C.

```c
#define _GNU_SOURCE
#include <stdio.h>
#include <dlfcn.h>          // dlsym()
#include <dirent.h>         // DIR, readdir
#include <string.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <sys/socket.h>
#include <limits.h>
#include <errno.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <netinet/tcp.h>
#include <sys/ioctl.h>
#include <termios.h>
#include <pty.h>
#include <signal.h>
#include <utmp.h>
```

### ✅ Explicação:

- `#define _GNU_SOURCE` ativa extensões GNU no compilador, necessário para funções como `execve`, `getline`, `fopen64`, etc.
    
- Múltiplas headers são usadas para:
    
    - Controle de terminal (`termios.h`, `pty.h`)
        
    - Manipulação de processos e arquivos (`stat.h`, `unistd.h`)
        
    - Rede e sockets (`arpa/inet.h`, `sys/socket.h`)
        
    - Gerenciamento de logs do sistema (`utmp.h`, `wtmp`


📚 **Documentação relevante**:

- [`dlsym()` - GNU libc
    
- `utmp` / `wtmp`

---

## 📁 Funções Analisadas

### `char *read_file(const char *file)`
#### O que faz:
Lê o conteúdo de um arquivo e retorna como uma string alocada dinamicamente.

#### Como funciona:

```c
fd = open(file, O_RDONLY);
```
- Abre o arquivo apenas para leitura.  
  📚 [man open(2)](https://man7.org/linux/man-pages/man2/open.2.html)

```c
fstat(fd, &s);
```
- Obtém informações do arquivo como tamanho.  
  📚 [man fstat(2)](https://man7.org/linux/man-pages/man2/fstat.2.html)

```c
buffer = malloc(s.st_size + 1);
read(fd, buffer, s.st_size);
buffer[s.st_size] = 0;
```
- Aloca um buffer, lê o conteúdo e adiciona `` ao final.

#### Por que faz isso:
Para ler arquivos sem depender de alto nível como `fgets` ou `fread`, com controle total do buffer.

---

### `int file_exists(const char *file)`
#### O que faz:
Verifica se um arquivo existe.

```c
stat(file, &s);
```
- Retorna 0 se o arquivo existe.  
  📚 [man stat(2)](https://man7.org/linux/man-pages/man2/stat.2.html)

---

### `int is_file_owned_by_euid(const char *file)`
#### O que faz:
Verifica se o arquivo pertence ao usuário efetivo do processo.

```c
getegid() == s.st_gid
```
- Compara o GID do arquivo com o GID efetivo do processo.  
  📚 [man 2 getegid](https://man7.org/linux/man-pages/man2/getegid.2.html)

---

### `char *get_input(const char *msg)`
#### O que faz:
Imprime uma mensagem e lê uma linha da entrada padrão.

```c
printf("%s", msg);
fgets(buf, sizeof(buf), stdin);
```

---

### `void trim(char *s)`
#### O que faz:
Remove caracteres de nova linha do final de uma string.

```c
s[strcspn(s, "
")] = 0;
```
- `strcspn` retorna o índice do primeiro `
` ou `
`.  
  📚 [man strcspn](https://man7.org/linux/man-pages/man3/strcspn.3.html)

---

### `void clearargv(char **argv)`
#### O que faz:
Sobrescreve os argumentos do processo na memória, útil para esconder comandos no processo.






---


# Guia para Desenvolvimento de Rootkit Userland em Linux (Tipo Azazel)

---

## Introdução

Este guia cobre os conceitos, técnicas e documentações necessárias para desenvolver um rootkit userland em Linux semelhante ao código Azazel, que intercepta syscalls, esconde arquivos/processos, implementa backdoor shell e técnicas anti-debug.

---

## 1. Hooking Dinâmico de Funções com LD_PRELOAD e dlsym

### Conceito

- Usar LD_PRELOAD para carregar uma biblioteca `.so` que sobrescreve funções libc.
- Chamar a função original com `dlsym(RTLD_NEXT, "funcao")` para evitar loop infinito.
- Interceptar chamadas para `open()`, `stat()`, `readdir()`, `accept()` e outras.

### Referências

- [man dlsym](https://man7.org/linux/man-pages/man3/dlsym.3.html)  
- [Tutorial LD_PRELOAD para Rootkits](https://null-byte.wonderhowto.com/how-to/hide-files-processes-linux-rootkit-using-ldpreload-0181779/)  
- [Blog Quarkslab: Hooking Linux Userland](https://blog.quarkslab.com/hooking-on-linux-part-1.html)

---

## 2. Syscalls Linux e Manipulação

### Conceitos

- Syscalls como `open()`, `stat()`, `access()`, `accept()`, `fork()`, `execve()` são alvo para esconder atividades.
- Entender como chamar syscalls originais dentro do hook para preservar funcionamento.

### Referências

- [Linux Syscall Table](https://filippo.io/linux-syscall-table/)  
- [man pages para syscalls (open, stat, access, accept)](https://man7.org/linux/man-pages/dir_section_2.html)  
- [Interceptando syscalls em C](https://www.ibm.com/docs/en/aix/7.2?topic=programs-intercepting-system-calls)

---

## 3. Manipulação de Terminais e PTYs

### Conceitos

- Criar pseudoterminais para shell interativo remoto (`openpty()`, `ioctl()`).
- Redirecionar entrada e saída do shell para socket remoto.
- Configurar sessão para shell usando `setsid()`, `dup2()`, `execve()`.

### Referências

- [openpty(3) manual](https://man7.org/linux/man-pages/man3/openpty.3.html)  
- [Artigo sobre Pseudo-terminals](https://www.linuxjournal.com/article/6420)  
- *Advanced Programming in the UNIX Environment* - Stevens (capítulo de terminal control)

---

## 4. Estruturas utmp e wtmp (Logs de Login)

### Conceitos

- Arquivos `/var/run/utmp`, `/var/log/wtmp` registram sessões e logins.
- Rootkit pode apagar entradas para esconder sessões.
- Uso da struct `utmp` e funções de leitura/escrita para manipular.

### Referências

- [man utmp(5)](https://man7.org/linux/man-pages/man5/utmp.5.html)  
- [Explicação sobre utmp/wtmp](https://unix.stackexchange.com/questions/10340/what-do-utmp-wtmp-and-btmp-logs-contain)  
- [Remoção de rastros de login](https://blog.rchapman.org/posts/Linux_System_Log_Files/)

---

## 5. Programação de Sockets e Multiplexação

### Conceitos

- Aceitar conexões TCP e criar shells interativos.
- Multiplexar I/O entre socket e pty com `select()`.
- Implementar autenticação simples para backdoor.

### Referências

- [Linux TCP/IP socket programming HOWTO](https://www.tldp.org/HOWTO/html_single/TCP-IP-Programming-HOWTO/)  
- [man select(2)](https://man7.org/linux/man-pages/man2/select.2.html)  
- [Beej’s Guide to Network Programming](http://beej.us/guide/bgnet/)

---

## 6. Técnicas Anti-Debug e Evasão

### Conceitos

- Bloquear chamadas `ptrace()` para impedir attach de debuggers.
- Evitar rastreamento por `ldd`, `ld-linux` e variáveis de ambiente de tracing.

### Referências

- [Anti-debugging no Linux](https://blog.filippo.io/the-boring-debugger/)  
- Técnicas de evasão para rootkits userland (diversos blogs de segurança)

---

## 7. Criptografia Simples e Backdoors

### Conceitos

- Uso de XOR simples para criptografar dados no canal do shell.
- Autenticação por senha simples na conexão.
- Implementação de shell reverso com PTY.

### Referências

- [XOR Encryption em C](https://www.geeksforgeeks.org/simple-xor-encryption-in-c-cpp/)  
- [Netcat/Ncat backdoors e shells](https://nmap.org/ncat/guide/)  
- [Criando um reverse shell com pty](https://medium.com/@varunon9/creating-a-linux-reverse-shell-with-pty-5089b5a36b3e)

---

## 8. Construção e Testes

### Dicas

- Compile a biblioteca compartilhada com:  
  ```bash
  gcc -shared -fPIC -o azazel.so azazel.c -ldl


```c
memset(argv[i], 0, strlen(argv[i]));
```

---

## 🔒 Considerações de Segurança

- Muitas dessas funções podem ser usadas para ofuscar atividades de um rootkit (ex: sobrescrever `argv`, verificar propriedade de arquivos, etc.)
- São implementadas manualmente para evitar dependências de bibliotecas que podem ser monitoradas.

---

## 📚 Documentações Recomendadas

- https://man7.org/
- https://en.cppreference.com/w/c
- https://www.gnu.org/software/libc/manual/

---

*Análise gerada automaticamente para fins educacionais.*
`
