# Tarefas da Semana #4 #

Nesta atividade é pedido aos alunos que compreendam de que forma as variáveis ambiente podem afetar o comportamento de um programa ou de um sistema.

Usadas por grande parte dos sistemas operativos desde a sua introdução ao Unix em 1979, as variáveis ambiente são um conjunto de valores dinâmicos que podem afetar a forma com que o processo se comportará no computador. 

A forma como estas variáveis afetam o comportamento do programa ainda não é bem compreendida pelos programadores e, como resultado, pode levar a que os programas tenham vulnerabilidades.


# Tarefa 1: Manipular Variáveis Ambiente

Nesta tarefa, analisar-se-á os comandos que podem ser utilizados para definir e desajustar as variáveis de ambiente.

- O comando `printenv`, ou apenas `env`, imprime as variáveis ambiente na consola como pode-se verificar [aqui](https://git.fe.up.pt/fsi/fsi2122/l04g02/-/blob/main/Week%204/task%201.1.txt)

- Pode-se usar o comando `export` para definir uma variável ambiente. O processo contrário pode ser obtido através do comando `unset`.
Para exemplicar este processo, criou-se uma variável **_a_** com o valor **_5_** e usou-se o comando `export` para a tornar numa variável ambiente. Usando o comando `printenv a` verifica-se que foi criada com sucesso. Usando o comando  `unset a` seguido do comando `printenv a`, verifica-se que a variável já não existe. Todos estes passos estão documentados abaixo: 

```
[11/10/21]seed@VM:~$  a=5  
[11/10/21]seed@VM:~$ export a
[11/10/21]seed@VM:~$ printenv a
5
[11/10/21]seed@VM:~$ unset a
[11/10/21]seed@VM:~$ printenv a
[11/10/21]seed@VM:~$ 
```

# Tarefa 2 : Passar Variáveis Ambiente do Processo Pai para o Processo Filho

Nesta tarefa, estuda-se como um processo filho obtém as variáveis ambiente do seu pai. Em Unix, fork() cria um novo processo ao duplicar o processo de chamada. Verificar-se-á se as variáveis ambiente do processo filho são herdadas ou não do processo pai.

- No primeiro passo, imprime-se as variáveis ambiente do processo filho: [(filho)](https://git.fe.up.pt/fsi/fsi2122/l04g02/-/blob/main/Week%204/task%202.1.txt)

- No segundo passo, faz-se o mesmo, mas para o processo pai: [(pai)](https://git.fe.up.pt/fsi/fsi2122/l04g02/-/blob/main/Week%204/task%202.2.txt)

- Por fim, compara-se os dois ficheiros gerados através do comando `diff`.

```
[11/10/21]seed@VM:~/WEEK #4$ diff task2.1.txt task2.2.txt 
[11/10/21]seed@VM:~/WEEK #4$
```

Uma vez que nada foi imprimido na consola, verifica-se que as variáveis ambiente do processo filho **são** herdadas do processo pai uma vez que as variáveis ambiente do processo pai são iguais ás do processo filho. 

# Tarefa 3 : Variáveis Ambiente e `execve ()`

Nesta tarefa, estuda-se a forma que as variáveis ambiente são afetadas quando um programa novo é executado via `execve()`. A função anterior, essencialmente, corre um programa dentro do processo de chamada. Pretende-se verificar se as variáveis ambiente são automaticamente herdadas pelo novo programa. 

- No primeiro passo, executa-se um programa que imprime as variáveis ambiente do processo atual. Como é possivel verificar abaixo, este programa não tem qualquer tipo de variáveis ambiente.

```
[11/10/21]seed@VM:~/WEEK #4$ gcc myenv.c
[11/10/21]seed@VM:~/WEEK #4$ ./a.out
[11/10/21]seed@VM:~/WEEK #4$
```

- No segundo passo e terceiro passo, ao alterar o ultimo argumento de `execve()` de `NULL` para `environ`, está-se a passar as variáveis ambiente do sistema ao novo programa e verifica-se [aqui](https://git.fe.up.pt/fsi/fsi2122/l04g02/-/blob/main/Week%204/task%203.2.txt) que este as imprime na consola.

É então possível concluir que as variáveis ambiente são herdadas automaticamente pelo novo programa.

# Tarefa 4 : Variáveis Ambiente e `system ()`

Nesta tarefa, compreender-se-á de que forma as variáveis ambiente são afetadas quando um novo programa é executado através da função `system()`.

Esta função difere da função `execve()` uma vez que esta última executa um comando diretamente, ao contário de `system()`, que pede à shell que execute tal comando.

Usando esta função, as variáveis ambiente do processo que a chama, são passadas ao novo programa, como pode-se verificar [aqui](https://git.fe.up.pt/fsi/fsi2122/l04g02/-/blob/main/Week%204/task%204.txt)

# Tarefa 5 : Variáveis Ambiente e Programas `Set-UID`

Set-UID é uma mecanismo de segurança importante em sistemas operativos Unix. Quando um programa deste tipo é executado, ele assume os privilégios do dono. Nesta tarefa, quer-se verificar se as variáveis ambiente do programa Set-UID são ou não herdadas do processo do utilizador.

- primeiro passo : criação de um programa que imprime as variáveis ambiente do processo atual

- segundo e terceiro passo : tornar o programa do tipo Set-UID e configurar algumas variáveis ambiente através do comando `export`. Correndo o programa Set-UID, verifica-se que as variáveis ambiente criadas no processo da shell (processo pai) passam para o processo filho Set-UID, à exceção da variável LD_LIBRARY_PATH, como pode-se observar [aqui](https://git.fe.up.pt/fsi/fsi2122/l04g02/-/blob/main/Week%204/task%205.txt)

# Tarefa 6 : A Variável Ambiente PATH e Programas `Set-UID`

Nesta tarefa demostrarse-á que invocar um programa shell dentro de um programa Set-UID é perigoso, uma vez o comportamento do programa shell pode ser aftado pelas variáveis ambiente e estas são fornecidas pelo utilizador, que pode não ter as melhores intenções.

De facto, foi possivel demostrar que é possível correr código malicioso através de um programa Set-UID, alterando a variável ambiente PATH, e criando um executável com o mesmo nome da chamada ao sistema implementada no programa : `ls`. Tal se verifica abaixo (`ls.c`, código fonte de um programa que imprime uma mensagem de alerta)

```
[11/10/21]seed@VM:~$ gcc task6.c -o task6
[11/10/21]seed@VM:~$ gcc ls.c -o ls
[11/10/21]seed@VM:~$ sudo chown root task6
[11/10/21]seed@VM:~$ sudo chmod 4755 task6
[11/10/21]seed@VM:~$ export PATH=/home/seed:$PATH
[11/10/21]seed@VM:~$ task6
Running mallicious code, be careful next time
[11/10/21]seed@VM:~$ 
```

Contudo, este programa não corre com os privilégios root, pois existe uma medida de segurança no Ubuntu que torna `/bin/sh` num link simbólico que aponta para `/bin/dash`, assim, se `dash` detetar que está a ser executado dentro de um programa Set-UID, os privilégios são retirados.

Por esse motivo, utiliza-se o comando: $ sudo ln -sf /bin/zsh /bin/sh, que é sugerido no tutorial, como forma de utilizar uma shell que não tivesse essa medida de segurança. Este comando liga `/bin/sh` a uma shell `/bin/zsh`, que não irá detetar a falha de segurança. Contudo, mesmo usando este comando, o ficheiro com código malicioso não corre com os privilégios root.
