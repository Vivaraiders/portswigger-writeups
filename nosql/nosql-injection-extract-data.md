# Exploiting NoSQL injection to extract data — PortSwigger Lab

**Dificuldade:** Practitioner
**URL:** https://portswigger.net/web-security/nosql-injection/lab-nosql-injection-extract-data
**Vulnerabilidade:** Página de lookup vulnerável a NoSQL
**Objetivo:** Extrair a senha do usuário `administrator` e logar na conta dele

## Como completar

Primeiro abri o Burp Suite e deixei na aba de HTTP History, e fiz login com o usuário `wiener/peter` para que eu pudesse analisar todas as requisições e respostas que iam e vinham da API. Fui analisando o caminho todo e alterando valores como `id=wiener` → `id=administrator` para entender como os valores estavam sendo tratados pela aplicação, porém quando alterei `user=wiener` → `user=administrator` recebi um JSON diferente na response:

**Antes:**
```json
{
  "username": "wiener",
  "email": "wiener@normal-user.net",
  "role": "user"
}
```

**Depois:**
```json
{
  "username": "administrator",
  "email": "admin@normal-user.net",
  "role": "administrator"
}
```

Foi aí que percebi que ali tinha uma possibilidade de que fosse um input vulnerável. Testei colocar uma aspa simples (`'`) para ver como a aplicação reagiria:

```
user=wiener'
```

E ele retornou:

```json
{
  "message": "There was an error getting user details"
}
```

Como retornou um erro com uma aspa simples (`'`), deduzi que o erro poderia estar vindo de uma possível concatenação do input com a query, algo como:

```
.find("this.user = '" + input_url + "'")
```

Então quando eu fechava a `'`, dava erro. Porém, nesse caso de procurar em query, o operador usado é o `$where`. Sabendo disso, pude tentar injetar um payload JS:

```
'||'1'=='1
```

Que ficou:

```
user=wiener'||'1'=='1
```

O que não retornou erro, e sim outro usuário — no caso, retornou dados do user `administrator`. Por quê? Não sei se retornou porque ele tem um id menor na coleção, ou se tem algum outro fator, mas o importante é que `1==1` retornou `true` e o nosso payload estava sendo interpretado pela aplicação.

Em seguida, sabendo que a JS injection funcionou e tem um `$where` por trás, pude pensar em como fazer para descobrir a senha do administrator.

**Payload**

```
user=administrator' && this.password[0]=='a
```

Ou melhor, encodado:

```
user=administrator' %26%26 this.password[0]=='a
```

(encodar o `&&` para a URL não interpretar como se tivesse outro parâmetro vindo, e sim uma condição JS)

Colocando esse payload no request e depois no Intruder, podemos adicionar o `'a'` (ficaria algo como `$a$`) e adicionar na wordlist as letras de a-z para que possa testar todos os caracteres no payload, obtendo a primeira letra da senha.

Se aparecer isso, a letra está errada:

```json
{
  "message": "There was an error getting user details"
}
```

E isso se tiver certo:

```json
{
  "username": "administrator",
  "email": "admin@normal-user.net",
  "role": "administrator"
}
```

Após isso, temos que ir mudando o número do índice do `this.password` sempre 1 número acima.

```
user=administrator' %26%26 this.password[1]=='$a$
user=administrator' %26%26 this.password[2]=='$a$
```

(esse `$a$` é adicionado pelo Intruder e não manualmente)

Depois é só pegar todos os caracteres que retornaram o `[username, email, role]` e usar como senha do administrator na página de login.

## O que aprendi

- Minha primeira tentativa foi reaproveitar `$ne` e `$regex`, porque tinham funcionado no lab anterior. Quando percebi que eles não produziam nenhum efeito, entendi que precisava considerar outro tipo de NoSQL Injection em vez de insistir na mesma técnica.
- Entendi a diferença entre operator injection e `$where` injection. Enquanto `$ne` e `$regex` alteram a estrutura da query, o `$where` permite injetar uma expressão JavaScript que será avaliada durante a consulta.
- Aprendi que o `$where` executa a condição para cada documento da coleção usando `this`, o que permitiu testar a senha do usuário `administrator` caractere por caractere.
- Também aprendi que operadores como `&&` precisam ser URL-encodados (`%26%26`) quando enviados pela URL, para que não sejam interpretados como separadores de parâmetros.
