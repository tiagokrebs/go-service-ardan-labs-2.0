A implementação de todos os conceitos desse módulo podem ser encontradas [aqui](https://github.com/tiagokrebs/go-app-2.0/blob/main/app/sales-api/main.go).

### Project setup

Estrutura de camadas sugerida:
- Um repositório por projeto (múltiplos binários)
- Diretório `app`: contém todo source da camada de aplicativo, binários, camada de apresentação, CLIs;
- Diretório `busines`: regras de negócio, source relativo ao propósito do projeto, específico ao problema ser resolvido, código não reutilizável, produz log;
- Diretório `foundation`: código reutilizável em diferentes projetos, provável projeto "kit" com repo exclusivo, espécie de standart library da companhia, não produz log;

Guidelines introdutórios:
- app/busines/foundations são vistos como packages independentes com regras específicas de design para cada um;
- Importações entre packages acontecem apenas no sentido botton-to-top;
- Alta granulação imports entre packages da mesma camada deve ser observada, epecialmente em busines. O cenário provável é mais favorável a menos packages de tamanho menor do que múltiplos pequenos packages;


### Logging

Definir o propósito dos logs deve ser uma das primeiras etapas do projeto.
- Os logs representam dados ou são para efeito de debug? Se data, há possibilidade de dar preferência para uso de métricas?
- Estruturados (para coleta, parse, storage) ou apenas human redable?
- Use a standad library para escrever log, não reinvente a roda, mantenha a consistência


### Configuration

- 100% de parâmetros com valor default válidos para ambiente de desenvolvimento
- Definicção de defaults entre ambientes diferentes (dev/staging/prod)
- Apenas main.go pode importar do sistema configuração passando aos packages via parâmetro
- Todo parâmetro deve permitir alteração via flags em linha de comando (--help)
- Todo parâmetro/override deve ser printado para garantir consistência
- Omitir log de parâmetros críticos como chaves e senhas é crítico


### Debugging amd metrics

- É possível obter da standard library métodos para disponibilizar valores de debug como métricas, através de uma rota específica. Importando `expvar` e `net/http/pprof` para esse fim não há adição de custo/latência na aplicação.
- O pacote `divan/expvarmon` é uma ferramenta para análise desses endpoints de debug quando coleta de métricas via Prometheus ou algum datadog não é necessário ou viável.


### Shutdown and Signaling
- Receber sinais para açõees como shutdown e reload de configuraçõe é necessário, especialmente quando a aplicação recebe uma quantidade significativa de requests ou uma grande quantidade de processamento crítico.
