# OS Command Injection — Simple Case

**Fonte:** https://portswigger.net/web-security/os-command-injection/lab-simple
**Categoria:** Command Injection
**Dificuldade:** Apprentice
**Objetivo:** Executar `whoami` na aplicação

## Contexto

A vulnerabilidade está na página de estoque de produto. Analisando o request no Burp, o que é enviado para a API é:

```
productId=1&storeId=1
```

## Identificação da vulnerabilidade

Para confirmar que a vulnerabilidade está ali e não em outra página, testei uma aspas simples (`'`) no parâmetro. A aplicação retornou:

```
sh: 1: Syntax error: Unterminated quoted string
```

**Por que isso confirma a vulnerabilidade:**
- Erro de sintaxe porque ficou uma string incompleta (faltando fechar a aspas)
- O `sh` no início do erro é o detalhe mais importante: indica que o input está sendo passado para um shell/terminal do sistema, ou seja, o parâmetro é interpretado como parte de um comando, não só como texto puro

## Exploração

Com a confirmação, é possível usar operadores de shell para encadear um comando próprio junto com o que a aplicação já envia:

```
productId=1&storeId=1;whoami
```
ou
```
productId=1&storeId=1|whoami
```

Ambos resolvem o lab.

## Operadores de shell usados

- **`;`** — roda os dois comandos em sequência, um depois do outro, independente do resultado do primeiro. Equivalente a uma quebra de linha entre os comandos.
- **`|` (pipe)** — pega a saída do comando à esquerda e usa como entrada do comando à direita. Como `whoami` não usa nenhuma entrada, ele simplesmente ignora o que vem do lado esquerdo e roda normalmente, retornando o usuário atual.

### Outros operadores relevantes (para referência)
- **`&&`** — roda o segundo comando somente se o primeiro tiver sucesso
- **`||`** — roda o segundo comando somente se o primeiro falhar

## Observação final

O resultado do `whoami` aparece na resposta porque a aplicação exibe a saída do último comando executado, e não porque houve alguma prioridade de "ordem de execução" do pipe em si.
