# Blind OS Command Injection with Time Delays — PortSwigger Lab

**Dificuldade:** Practitioner
**URL:** <https://portswigger.net/web-security/os-command-injection/lab-blind-time-delays>
**Vulnerabilidade:** A página de feedback é vulnerável a OS command injection, porém não sabemos em qual input
**Objetivo:** Fazer com que a aplicação demore 10 segundos para retornar a resposta

## Como completar

Antes desse lab, em outras categorias como SQLi, resolvi labs que exigiam causar um delay grande entre requisição e resposta, então já sabia que a lógica passaria por um `sleep`. Pesquisei antes se Linux e Unix operam com essa mesma palavra-chave para delay.

Para confirmar que a vulnerabilidade estava ali e não em outro input, testei uma aspa simples (`'`) no parâmetro. No input de e-mail, a aplicação retornou:

```
Could not save
```

Isso levantou a hipótese de que a aplicação estava rodando algum código no terminal do servidor para salvar o e-mail, e a aspa quebrou a sintaxe esperada, invalidando o comando.

## Payload

Com essa hipótese confirmada, removi a aspa simples e usei o operador `;`, que serve como indicação para a máquina que está lendo o terminal do servidor (real ou virtual) de que ali se encerra o comando de e-mail e se inicia um comando diferente.

Payload utilizado:

```
email=teste%40gmail.com; sleep 5;
```

O `;` no final também indica que o comando deve rodar até ali, sem passar parâmetros inexistentes que poderiam atrapalhar a execução (ex: o `sleep` recebendo um argumento inválido colado logo depois dele).

Após confirmar o delay de 5 segundos, ajustei o payload para bater o objetivo exato do lab:

```
email=teste%40gmail.com; sleep 10;
```

Delay de 10 segundos confirmado na resposta da requisição via Burp Repeater.

## O que aprendi

- Testar caracteres de quebra de sintaxe (`'`, `"`, `;`) é uma boa primeira etapa para localizar o input vulnerável antes de montar o payload final
- Isolar o comando injetado com `;` nos dois lados (`;comando;`) evita que argumentos residuais do comando original quebrem a execução
- Confirmar a vulnerabilidade com um delay pequeno (`sleep 5`) antes de ir para o valor exato pedido pelo lab é mais seguro do que já mandar o valor final
