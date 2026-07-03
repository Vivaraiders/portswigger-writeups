# Exploiting NoSQL operator injection to bypass authentication — PortSwigger Lab

**Dificuldade:** Apprentice
**URL:** https://portswigger.net/web-security/nosql-injection/lab-nosql-injection-bypass-authentication
**Vulnerabilidade:** Página de login vulnerável a operadores de NoSQL
**Objetivo:** Acessar o perfil admin

## Como completar

Sabendo que a vulnerabilidade é injeção de operadores no JSON de autenticação, a primeira coisa que fiz foi pesquisar sobre os operadores e suas funcionalidades. Dentre eles, teve um que me chamou bastante atenção que é o `$ne`, que significa o equivalente à `!=` (diferente).

Ele basicamente retorna um valor diferente do que você passar: `{"$ne":"123"}` → se for diferente de 123 será retornado.

Como eu recebi o email e a senha `wiener/peter`, coloquei cada valor em seu respectivo input, liguei o Burp e fui analisar o body da requisição no Repeater. Primeiro testei o operador `$ne` no usuário:

```json
{
  "username": "wiener",
  "password": {
    "$ne": ""
  }
}
```

*obs: significa que se for algum valor diferente de uma string vazia na senha e que o usuário seja wiener*

Analisando a resposta da API vi que funcionou, logava no usuário wiener. Mas como o objetivo era acessar o usuário admin, coloquei `$ne` na senha e no usuário coloquei `administrator`:

```json
{
  "username": "administrator",
  "password": {
    "$ne": ""
  }
}
```

Recebi a seguinte mensagem no response:

```
<p class=is-warning>Invalid username or password</p>
```

O que não era pra acontecer SE o usuário realmente fosse `administrator`. Foi quando comecei a pensar: "como vou descobrir qual realmente é o username do perfil admin?". Procurei alguma dica na descrição, mas só falava que era pra acessar o perfil administrador. Aí voltei pra estaca zero de novo, fui ver os passos que fiz, e me deparei novamente com o título do lab, que diz que a vulnerabilidade é injeção de operador.

Voltei a pesquisar sobre operadores e me deparei com `$regex`, que parecia ser exatamente o que eu precisava. Então testei o seguinte payload:

**Payload**

```json
{
  "username": {
    "$regex": "^adm"
  },
  "password": {
    "$ne": ""
  }
}
```

*obs: regex serve para retornar valores que começam, terminam ou que contêm algum valor (de acordo com o que o caractere especial significa). O `^` significa que começa com "adm", poderia ser algo como `['admin', 'administrador', 'admin123']` ou qualquer outra variação que comece com essa sequência. Só tem que ficar atento porque se tivesse um perfil chamado "admil" (ou qualquer outro perfil que comece com "adm"), ele seria o primeiro a dar match e consequentemente seria logado nele.*

O response dessa query retornou:

```
HTTP/2 302 Found
Location: /my-account?id=admin6mb0hztv
Set-Cookie: session=xFhbQxmz2Ze1c2AxcjVEmOT79aiSH5ik; Secure; HttpOnly; SameSite=None
X-Frame-Options: SAMEORIGIN
Content-Length: 0
```

O que significa que o regex achou o username `admin6mb0hztv` e a senha era diferente de uma string vazia, e vai me direcionar. Código 302 significa que funcionou e será redirecionado, o `Location` é para onde irá: `/my-account?id=admin6mb0hztv`.

## O que aprendi

- Entendi que quando a aplicação espera uma string e não valida isso direito, dá pra mandar um objeto JSON no lugar (tipo `{"$ne": ""}` ou `{"$regex": "^adm"}`) e o MongoDB simplesmente aceita isso como operador de query em vez de tratar como valor literal.
- O `$ne` resolve o problema de não saber o user ou a senha, e pode ser muito bem usado junto com o `$regex` quando você não sabe nem o user nem a senha, mas tem uma ideia de como seja uma parte de algum dos dois campos.
- `$regex` com match parcial (tipo `^adm`) tem um risco: se houver mais de um perfil que bata com esse prefixo, o banco pode retornar o primeiro que encontrar, e eu não teria como saber se estou logando no perfil certo. Pra evitar isso, dá pra usar `^adm$` quando já souber o valor exato, ou enumerar caractere por caractere até fechar o username completo.
- Esse tipo de vulnerabilidade não tem nada a ver com SQL injection tradicional, então defesas feitas só pra SQL (tipo escapar aspas) não adiantam aqui. O certo é validar o tipo do dado antes de montar a query, garantindo que username e senha sempre sejam string.
