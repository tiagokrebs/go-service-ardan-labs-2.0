### Module mirrors

Estudar a semântica de módulos não se resume a simples necessidade para egenharia de APIs mas também na tomada de decisões relacionadas a privacidade que vão acontecer durante as etapas de projeto e construção.

Na versão atual (1.15) módulos não precisam mais da variável de ambiente `GOPATH`. Entretanto algumas vezes é importante observar o cache dentro desse path, realizando a limpeza quando necessário (mais detalhes a frente).
```
go env | grep GOPATH 
GOPATH="/Users/tiago.krebs/go"

# Go modules cache sofrem limpeza com mais frequência
go clean -modcache
rm -rf /Users/tiago.krebs/go/pkg/sumdb

# Pre-build Go packages cache dificilmente precisa ser apagado
go env | grep GOCACHE
GOCACHE="/Users/tiago.krebs/Library/Caches/go-build"
```

Definimos o uso de módulos em um projeto através de `go mod init`. É importante utilizar um nome único, diferente de qualquer outro disponível, não apenas localmente mas, se há intenção de compartilhar o projeto com a comunidade Golang, de forma global. Comumente o path do projeto no girhub é utilizado.
```
go mod init github.com/tiagokrebs/go-app-2.0
```
Dessa forma a mecânica de módulos passa a ser o método default do projeto para encontrar sources codes. O arquivo `go.mod` armazena a versão atual do Go, através desse arquivo algumas ferramentas built-in do Go são capazes de ajustar seu comportamento de acordo. 

Ao importar packages esse arquivo precisa ser atualizado com o nome do módulo e versão, fazemos isso através de `go mod tidy`. A escolha da versão cabe ao MVS.
```
package main

import (
	"log"
	"os"
	"github.com/pkg/errors"
)

func main() {
	if err := run(); err != nil {
		log.Println(err)
		os.Exit(1)
	}
}

func run() error {
	if 1 == 2 {
		return errors.New("randam error")
	}
	return nil
}
```
```
go mod tidy
go: finding module for package github.com/pkg/errors
go: found github.com/pkg/errors in github.com/pkg/errors v0.9.1
```

Nesse exemplo o módulo `github.com/pkg/errors` foi encontrado no app e registrado em `go.mod` utilizando a versão stable disponível no repositório, v0.9.1. Consultando o cache no `GOPATH`:
```
tree /Users/tiago.krebs/go/pkg/mod/github.com/   
/Users/tiago.krebs/go/pkg/mod/github.com/
└── pkg
    └── errors@v0.9.1
        ├── LICENSE
        ├── Makefile
        ├── README.md
        ...
```

Os imports são realizados com base na variável de ambiente `GOPROXY`. O proxy server, também conhecido como module mirror, é o endpoit adminstrado pelo Google que realiza o serviço de procura e download dos módulos importados. A diretiva `direct` é uma espécie dee fallback caso o mirror não possua o módulo cacheado.
```
go env | grep GOPROXY
GOPROXY="https://proxy.golang.org,direct"
```
Há um problema fundamental de privacidade nesse caso, apesar da política de privacidade do Google, os IPs que fizeram requisições vão ser registrados assim como os módulos. Nesse caso `GOPROXY` pode ser alterado, por exemplo utilizando apenas a diretiva `direct`, porém, há um novo problema nesse caso. Obter o source diretamente do github está sujeito a alterações constantes e a remoção do source (o que compromete a durabilidade e estabilidade do projeto).

A solução proposta é utilizar um Prived Module Mirror como o [Athens](https://docs.gomods.io/) ou [JfrogArtifactory](https://jfrog.com/artifactory/). Dessa forma basta apontar `GOPROXY` para o endpoint privado, mantendo `direct` como fallback.

A variável de ambiente `GONOPROXY` define quais módulos não devem ser obtidos através do Module Mirror.
```
go env | grep GONOPROXY 
GONOPROXY="me.gitlab.com"
```
Nesse exemplo todo source iniciando com `me.gitlab.com` será importado diretamente. Dessa forma código prioritário pode ser importado diretamente de um endpoint prioritário. Esse comportamento também pode ser criado diretamente no Athens ou JfrogArtifactory.

### MVS algorithm
