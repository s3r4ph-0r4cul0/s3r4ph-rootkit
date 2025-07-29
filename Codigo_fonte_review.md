
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
