# Blind OS Command Injection with Output Redirection — PortSwigger Lab

**Dificuldade:** Practitioner
**URL:** <https://portswigger.net/web-security/os-command-injection/lab-blind-output-redirection>
**Vulnerabilidade:** A página de feedback é vulnerável a OS command injection, porém não sabemos em qual input
**Objetivo:** Executar o comando `whoami` e ler seu output

## Como completar

Antes desse lab, em outras categorias como SQLi, resolvi labs que exigiam causar um delay grande entre requisição e resposta, então já sabia que a lógica passaria por um `sleep`. Pesquisei antes se Linux e Unix operam com essa mesma palavra-chave para delay.

Para confirmar que a vulnerabilidade estava ali e não em outro input, testei uma aspa simples (`'`) no parâmetro. No input de e-mail, a aplicação retornou:

```
Could not save
```

Isso levantou a hipótese de que a aplicação estava rodando algum código no terminal do servidor para salvar o e-mail, e a aspa quebrou a sintaxe esperada, invalidando o comando.

## Payload

Com essa hipótese confirmada, removi a aspa simples e usei o operador `;`, que serve como indicação para a máquina que está lendo o terminal do servidor (real ou virtual) de que ali se encerra o comando de e-mail e se inicia um comando diferente.

Payload utilizado para confirmar a vulnerabilidade:

```
email=teste%40gmail.com; sleep 2;
```

O `;` no final também indica que o comando deve rodar até ali, sem passar parâmetros inexistentes que poderiam atrapalhar a execução (ex: o `sleep` recebendo um argumento inválido colado logo depois dele).

Após confirmar o delay de 2 segundos, ajustei o payload para o objetivo real do lab — ao invés de só confirmar execução, redirecionar o output de um comando para dentro de uma pasta gravável e publicamente acessível pelo servidor web:

```
email=teste%40gmail.com; whoami > /var/www/image/whoami.txt;
```

Obs: o operador `>` faz com que a saída do comando à esquerda seja salva no caminho à direita.

Depois disso, voltei à home do PortSwigger, abri um produto qualquer e percebi que a imagem demorou um pouco pra aparecer após o carregamento da página — então deduzi que a imagem estava sendo servida a partir de algum caminho no servidor. Liguei o proxy do Burp e dei reload na página; após o carregamento, apareceu uma requisição de imagem (como esperado). Enviei essa requisição pro Repeater e mudei o parâmetro para o mesmo caminho onde salvei a saída do `whoami`:

```
/image?filename=whoami.txt
```

Com isso, consegui ler o conteúdo salvo nesse arquivo, ou seja, o output do comando `whoami` executado no servidor.

## O que aprendi

- O operador `>` do shell permite redirecionar a saída de um comando injetado para dentro de um arquivo — útil quando a injeção não retorna o resultado diretamente na resposta HTTP
- Se essa pasta estiver dentro do diretório público do servidor web (ex: `/var/www/`), o arquivo criado pode ser lido depois via requisição HTTP normal, transformando uma injeção "cega" em uma injeção com output legível
- Vale prestar atenção em pequenos detalhes de comportamento da aplicação (como um delay no carregamento de uma imagem) — eles podem indicar como e onde a aplicação busca arquivos no servidor
