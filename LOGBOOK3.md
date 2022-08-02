# Tarefas da Semana #3 #

**CVE-2002-0308**

## Identificação

- Tipo do produto - Aplicação
- Nome do produto - AdMentor (Versão 2.11 e anteriores)
- Tipo de ataque - Injeção SQL
- Descrição - É possivel a um atacante remoto ter acesso à página administrativa do software, através de um ataque do tipo injeção SQL.

## Catalogação

- Nível de gravidade - Máximo (CVSS Score - 10.0)
- Impacto na confidencialidade - Todos os ficheiros do sistema são revelados, assim como a informação dos utilizadores. 
- Impacto na integridade - O sistema fica totalmente comprometido
- Autenticação - Não é necessária

## Exploit

- Ao colocar no login: **' or ''='** e, na palavra passe **' or ''='** é possível criar uma interrogação válida à base de dados, uma vez que esta vai ser anexada na forma: 

`SELECT row from table where login = '' or ''=''`

- O mesmo se aplica à palavra passe.
- Através da processo anterior, conseguimos fazer login como administrador principal, sem qualquer problema.

## Ataques

- A vulnerabilidade foi reportada no momento da sua descoberta, através de um [email](https://marc.info/?l=bugtraq&m=101430885516675&w=2) em fevereiro de 2002, pelo utilizador thran(thran60@hotmail.com), tendo este oferecido uma solução temporária que removia carateres inválidos através de uma função em JavaScript.

