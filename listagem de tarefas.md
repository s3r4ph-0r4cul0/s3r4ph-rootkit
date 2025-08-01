# 📋 LISTA NUMERADA DE ANÁLISE DOS ARQUIVOS
**�� PRIORIDADE CRÍTICA (Análise Imediata)**

1. azazel.c (19KB, 821 linhas)
Por que primeiro: Arquivo principal do rootkit
O que analisar:
Função azazel_init() - inicialização
Hooks de system calls (open, stat, readdir, etc.)
Função is_invisible() - lógica de ocultação
Função drop_shell() - backdoor principal
Função clean_utmp/clean_wtmp() - limpeza de logs

---



2. const.h (4.5KB, 85 linhas)
Por que segundo: Define todas as constantes e configurações
O que analisar:
Portas de backdoor (LOW_PORT, HIGH_PORT, etc.)
Strings mágicas para ocultação
Configurações de criptografia
Tabela de system calls
3. azazel.h (1.6KB, 50 linhas)
Por que terceiro: Headers principais e estruturas
O que analisar:
Declarações de funções
Estrutura s_syscalls
Atributos de visibilidade
🟡 PRIORIDADE ALTA (Análise Essencial)
4. config.py (4.7KB, 127 linhas)
Por que quarto: Gerador de configuração
O que analisar:
Função xor() - ofuscação de strings
Configurações de portas e senhas
Geração automática do const.h
5. crypthook.c (4.5KB, 194 linhas)
Por que quinto: Sistema de criptografia
O que analisar:
Funções crypt_read/crypt_write
Implementação AES-256-GCM
Geração de chaves PBKDF2
6. pam.c (3.4KB, 145 linhas)
Por que sexto: Backdoor PAM
O que analisar:
Função pam_authenticate()
Função getpwnam()
Lógica de bypass de autenticação
�� PRIORIDADE MÉDIA (Análise Importante)
7. Makefile (726B, 30 linhas)
Por que sétimo: Sistema de build
O que analisar:
Flags de compilação
Dependências
Processo de instalação
8. pcap.c (1.6KB, 61 linhas)
Por que oitavo: Hooks de rede
O que analisar:
Função pcap_loop()
Ocultação de tráfego de rede
Estruturas de headers TCP/IP
9. pcap.h (1.7KB, 55 linhas)
Por que nono: Headers de rede
O que analisar:
Estruturas sniff_ip e sniff_tcp
Definições de flags TCP
Macros de manipulação
�� PRIORIDADE BAIXA (Análise Complementar)
10. xor.c (123B, 9 linhas)
Por que décimo: Função de ofuscação simples
O que analisar:
Função x() - XOR com chave 0xFE
Simplicidade da implementação
11. xor.h (54B, 7 linhas)
Por que décimo primeiro: Header da função XOR
O que analisar:
Declaração da função x()
12. crypthook.h (298B, 11 linhas)
Por que décimo segundo: Header de criptografia
O que analisar:
Declarações de crypt_read/crypt_write
Constante MAX_LEN
13. client.c (780B, 39 linhas)
Por que décimo terceiro: Cliente para backdoor PAM
O que analisar:
Hook da função socket()
Configuração de porta 61061
⚪ PRIORIDADE MÍNIMA (Análise Informativa)
14. README.md (1.6KB, 31 linhas)
Por que décimo quarto: Documentação do projeto
O que analisar:
Descrição das funcionalidades
Instruções de uso
Avisos legais
15. LICENSE (18KB, 340 linhas)
Por que décimo quinto: Licença GPL v2
O que analisar:
Restrições de uso
Requisitos de distribuição
Implicações legais
