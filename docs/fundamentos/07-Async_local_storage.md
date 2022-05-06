# Armazenamento local assíncrono
De acordo com a [documentação oficial do Node.js](https://nodejs.org/docs/latest-v14.x/api/async_hooks.html): "**AsyncLocalStorage** é usado para criar estado assíncrono dentro de callbacks e cadeias de promises. Ele permite armazenar dados durante toda a vida útil de uma solicitação da Web ou qualquer outra duração assíncrona. É semelhante ao armazenamento local de thread em outras linguagens."

Para simplificar ainda mais a explicação, `AsyncLocalStorage`permite armazenar um estado e disponibilizá-lo para todos os caminhos do códig dessa função. Por exemplo:

> O exemplo a seguir é imaginário. No entanto, você ainda pode continuar criando um projeto Node.js vazio.

Vamos criar uma instância de `AsyncLocalStorage` e exportá-la de seu módulo. Isso permitirá que vários módulos acessem a mesma instância de armazenamento.
```js
// storage.js
import { AsyncLocalStorage } from 'async_hooks'
export const storage = new AsyncLocalStorage()
```
Crie o arquivo principal. Ele usuará o método `storage.run` para executar uma função assíncrona com o estado inicial.
```js
// main.ts
import { storage } from './storage'
import ModuleA from './ModuleA'

async function run(id) {
  const state = { id }

  return storage.run(state, async () => {
    await (new ModuleA()).run()
  })
}

run(1)
run(2)
run(3)
```
Finalmente, `ModuleA` pode acessar o estado usando o método `storage.getStore()`.
```js
// ModuleA.ts
import { storage } from './storage'
import ModuleB from './ModuleB'

export default class ModuleA {
  public async run() {
    console.log(storage.getStore())
    await (new ModuleB()).run()
  }
}
```
Assim como o `ModuloA`, o `ModuloB` também pode acessar o mesmo estado usando o método `storage.getStore`.

Em outras palavras, toda a cadeia de operações tem acesso ao mesmo estado inicialmente definido dentro do arquivo `main.js` durante a chamada do método `storage.run`

## Qual é a necessidade de armazenamento local assíncrono?
Ao contrário de outras linguagens como PHP, o Node.js não é uma linguagem encadeada.

No PHP, cada solicitação HTTP cria uma nova thread, e cada thread tem sua memória. Isso permite que você armazene o estado na memória global e acesso-o em qualquer lugar dentro de sua base de código.

No Node.js, você não pode salvar dados em um objeto global e mantê-los isolados entre solicitações HTTP. Isso é impossível porque o Node.js é executado em um único thread e compartilha a memória em todoas as solicitações HTTP.

É aqui que o Node.js ganha muito desempenho, pois não precisa inicializar o aplicativo HTTP.

No entanto, isso também significa que você precisa passar o estado como argumentos de função ou argumentos de classe, pois não pode gravá-lo no objeto global. Algo como:
```js
http.createServer((req, res) => {
  const state = { req, res }
  await (new ModuleA()).run(state)
})

// Module A
class ModuleA {
  public async run(state) {
    await (new ModuleB()).run(state)
  }
}
```
> "O armazenamento local assíncrono aborda esse caso de uso, pois permite o estado isolado entre várias operações assíncronas."

## Como o AdonisJS usa o ALS?
ALS significa **AsyncLocalStorage**, se acostume com essa sigla. O AdonisJS usa o ALS durante as solicitações HTTP e define o [contexto HTTP](https://docs.adonisjs.com/guides/context) como estado. O fluxo de código é semelhante ao seguinte.
```js
storage.run(ctx, () => {
  await runMiddleware()
  await runRouteHandler()
  ctx.finish()
})
```
O middleware e o manipulador de rotas geralmente executam outras opções. Por exemplo, usando um modelo para buscar os usuários.
```js
export default class UsersController {
  public index() {
    await User.all()
  }
}
```
As instâncias do modelo `User` agora têm acesso ao contexto, pois são criadas no caminho do código do método `storage.run`.
```js
import HttpContext from '@ioc:Adonis/Core/HttpContext'

export default class User extends BaseModel {
  public get isFollowing() {
    const ctx = HttpContext.get()!
    return this.id === ctx.auth.user.id
  }
}
```
As propriedades estáticas do modelo (não métodos) não podem acessar o contexto HTTP, pois são avaliadas ao importar o modelo. Portanto, você deve entender o caminho de execução do código e usar o ALS com cuidado.

## Usos
Para usar o ALS em suas aplicações, você deve habilitá-lo primeiro dentro do arquivo `config/apps.ts`. Sinta-se à vontade para criar a propriedade manualmente se ela não existir.
```js
// config/app.ts
export const http: ServerConfig = {
  useAsyncLocalStorage: true,
}
```
Uma vez habilitado, você pode acessar o contexto HTTP atual em qualquer lugar dentro do seu código usando o módulo `HttpContext`.

> Certfique-se de que o caminho do código seja chamado durante a solicitação HTTP para que o `ctx` esteja disponível. Caso contrário, será nulo.

```js
import HttpContext from '@ioc:Adonis/Core/HttpContext'

class SomeService {
  public async someOperation() {
    const ctx = HttpContext.get()
  }
}
```
## Como deve ser usado?
Nesse ponto, você pode considerar o ALS como um estado global específico da solicitação. O [estado ou variáveis globais são geralmente considerados ruins](https://wiki.c2.com/?GlobalVariablesAreBad), pois dificultam muito o teste e o debugging.

O ALS do Node.js pode ficar ainda mais complicado se você não for cuidadoso o suficiente para acessar o ALS da solicitação HTTP.

Recomendamos que você ainda escreve seu código como estava escrevendo anteriormente (passando o `ctx` por referência), mesmo que tenha acesso ao ALS. A passagem de dados por referência transmite um caminho de execução claro e facilita o teste de seu código isoladamente.

### Então, por que você introduziu o ALS?
O armazenamento local assíncrono se destaca com as ferramentas de APM, que coletam métricas de desempenho do seu aplicativo para ajudá-lo a debugar e identificar problemas.

Antes do ALS, não havia uma maneira simples de as ferramentas APM relacionarem diferentes recursos com a determinada solicitação HTTP. Por exemplo, ele pode mostrar o tempo gasto por uma determinada consulta SQL, mas não pode informar qual solicitação HTTP executou essa consulta.

Depois do ALS, tudo isso agora é possível sem que você ajuste uma única linha de código. O AdonisJS usuará o ALS para coletar métricas usando seu perfilador de nível de aplicativo.

## Coisas para ficar esperto ao usar ALS
Você está livre para usar o ALS se achar que isso torna seu código mais direto e preferir acesso global ao passar tudo por referência.

No entanto, esteja ciente das seguintes situações que geralmente podem levar a vazamentos de memória ou comportamento instável do programa.

### Acesso no Top-level
Nunca acesse o ALS no topo do seu código. Por exemplo:

#### ❌ Não funciona
No Node.js, os módulos são armazenados em cache. Portanto, o método `HttpContext.get()` será executado apenas uma vez durante a primeira solicitação HTTP e manterá seu `ctx` para sempre durante o ciclo de vida do seu processo.
```js
import HttpContext from '@ioc:Adonis/Core/HttpContext'
const ctx = HttpContext.get()

export default class UsersController {
  public async index() {
    ctx.request
  }
}
```
#### ✅ Funciona
Em vez disso, você deve mover para a chamada `.get` dentro do método `index`.
```js
export default class UsersController {
  public async index() {
    const ctx = HttpContext.get()
  }
}
```
### Por dentro das propriedades estáticas
As propriedades estáticas (não métodos) de qualquer classe são avaliadas assim que o módulo é importado e, portanto, você não deve acessar o `ctx` dentro das propriedades estáticas.

#### ❌ Não funciona
No exemplo a seguir, quando você importa o modelo `User` dentro de um controller, o código `HttpContext.get()` será executado e armazenado em cache para sempre. Portanto, você receberá `null` ou acabará armazenando em cache a conexão do locatório da primeira solicitação.
```js
import HttpContext from '@ioc:Adonis/Core/HttpContext'

export default class User extends BaseModel {
  public static connection = HttpContext.get()!.tenant.connection
}
```
#### ✅ Funciona
Em vez disso, você deve mover a chamada `HttpContext.get()` para dentro do método de consulta.
```js
import HttpContext from '@ioc:Adonis/Core/HttpContext'

export default class User extends BaseModel {
  public static query() {
    const ctx = HttpContext.get()!
    return super.query({ connection: tenant.connection })
  }
}
```
### Manipuladores de eventos
O manipulador de um evento emitido durante uma solicitação HTTP pode obter acesso ao contexto da solicitação usando o método `HttpContext.get()`. Por exemplo:
```js
export default class UsersController {
  public async index() {
    const user = await User.create({})
    Event.emit('new:user', user)
  }
}
```
```js
// Event handler
import HttpContext from '@ioc:Adonis/Core/HttpContext'

Event.on('new:user', () => {
  const ctx = HttpContext.get()
})
```
No entanto, você deve estar ciente de algumas coisas ao acessar o contexto de um manipulador de eventos. Por exemplo:

- O evento nunca deve tentar enviar uma resposta usando `ctx.response.send()` porque não é isso que os eventos devem fazer.
- Acessar o `ctx` dentro de um manipulador de eventos faz com que ele dependa de solicitações HTTP. Em outras palavras, o evento não é mais genérico e deve sempre ser emitido durante uma requisição HTTP para que funcione.

<footer align="center">
  <a href="./06-Deployment.md">Voltar para Deployment</a>
</footer>
