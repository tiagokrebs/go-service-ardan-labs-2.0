### Custom handler
Extensão do httpmux atual, construção de um framework simples na camada foundation (`foundation/web`) para adição de boilerplates e consistência no projeto. Isso é feito atraés de embedding do httpmux no novo package web e então a troca de toda utilização pelo source para esse package.  
A motivação principal é a escrita de um método `Handler` personalizado para ser usado no lugar do httpmux handler, o que traz mais flexibilidade ao projeto. consistência na utilização pelas rotas, error handnling, uso de middlewares e monitoração de sinais enviados (ex: shutdown).

### Gathering trace information
O handler criado nos dá a capacidadse de adicionar código antes e depois do tratamento da requisição efetuada nas rotas. Logging, error handling, coleta de métricas, panic handling são alguns exemplos do que pode ser utilizado, exceto pelo fato que um package dentro de foundation não pode escrever log de acordo com o nosso guideline para esse projeto. Isso é resolvido mais tarde através das regras de business, um middleware e téeclica de log trace id.  

O http context na camada de busines deve estar vazio, utilizá-lo para técnica de logginng e error handling adiciona uma nova camada de complexidade nos testes e debug da aplicação. Foundation utiliza a camada única e exclusivamente com objetivo de tracing e debug.

### Middleware
Handler que retorna um handler, como um onion factory ou um handler encapsula uma function e adiciona código antes e depois da sua execução. 
O middleware desse app é um handler que encapsula outro handler e adicona código antes e depois, criando várias camadas.
É comum que e cada camada do midleware tenha um objetivo específicio no app gerando um trajeto padrão para todo o sistema. Exemplo seguindo da camada mais interna (regra de negócio) para externa: readines -> authentication -> panic -> metric -> errors -> logging.
Devido a característica do moddleware cada camada é escrica como uma regra de negócio montando a fundação para o framework do app.

### Error handling
Evita handling dos erros nas regras de negócio trazendo para uma camada do middleware. Três conceitos importantes:
- O erro deve parar o código no momento do handling, não propaga para stack
- O erro deve ser escrito em log com o contexto compledo da stack
- Deve ocorrer por decisão se o app deve sofrer shutdown ou se um restore deve ocorrer
A camada de foundation não pode conter error handling, porque não pode escrever log. Camada de regras de negócio e aplicação podem fazer error handling e enviar sinais para um possível shutdown.
`%w` do package fmt possui métodos para obter o contexto através do error package interno porém ela não é utilizada no app. O package não atende o formato esperado para o app construído aqui.
Error values podem estar na camada de fundation ou regras de negócio. Nesse projeto a camda fundation é utilizada, dessa forma os valores podem ser normalizados entre diferentes apps que utlizam os packages dessa camada.

### Sending responses
Handling de responses é utilizado para garantir a consistência e abstração do contexto. Garantia de tipos e validação da existência de valores no contexto é feita nessa camada.

### Handling requests
Na última etapa da construção do web framework adicionamos validação dos dado do request, unmarshal, validação de JSON, etc.
Init() é para inicialização de comportamento do package não para instanciação de variáveis ou contrução de structs.