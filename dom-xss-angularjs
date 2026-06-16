LAB: DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded

como cheguei a solução:
Lendo a descrição do lab percebe-se que é uma aplicação em AngularJS(pelo uso de ng-app) e que é possivel executar comandos dentro de chaves duplas. Com isso primeiro
dei uma analisada no codigo fonte pra ver se tinha algo interessante por ser o primeiro lab de angular me senti perdido, porém como a descrição dizia sobre executar JS
dentro de chaves duplas, testei: '{{7*7}}' e percebi que o resultado veio '49', testei em seguida um 'alert()' e não foi o suficiente, então fiz uma pesquisa sobre
como eu resolveria esse lab, me deparei com 'AngularJS sandbox escape alert' e entendi que há uma maneira de chamar uma função sem chamar diretamente a palavra reservada 
'function', e sim, 'this.constructor.constructor' que seria a forma bruta de como uma function é feita, com isso desenvolvi o payload:
 '{{this.constructor.constructor('alert(1)')()}}' e o lab foi resolvido.

esplicação completa do payload:
quando vc usar o 'this.constructor', o que isso significa: quando você está no escopo central, o construtor é o 'Object', sabendo disso quando, vc usa o 'this.constructor',
está se referindo a ele, em seguida vem mais um'.constructor' que retrata o construtor do object: 'function'. Então a grosso modo você está basicamente fazendo o caminho 
completo que o JS faz quando cria uma função
