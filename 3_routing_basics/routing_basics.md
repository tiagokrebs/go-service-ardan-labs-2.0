
A implementação de todos os conceitos desse módulo podem ser encontradas [aqui](https://github.com/tiagokrebs/go-app-2.0/tree/main/app/sales-api/handlers).

### Handler package

- Handler para as rotas da aplicação são permitidas a: receber requests e realizar validações, dar procedimento com as regras da camada busines, formular resposta OK
- Rotas não fazem hanlder de erros
- O HTTP-mux é utilizado para disponibilizar as rotas


### Readiness handler
- Readiness é um endpoint dedicado a ambientes que utilizam um sistema orquestrado (containers/kubernetes) que informa quando a aplicação está pronta para receber requests
- Liveness tem um conceito semelhante porém normalmente é dedicado a mensagems alive em kubernetes
- 