# Application - Aplicação

O módulo aplicativo do AdonisJS é responsável por inicializar o app em diferentes ambientes conhecidos.

Quando você inicia o server HTTP do arquivo `server.ts` ou executa o comando `node ace serve`, a aplicação é inicializada para um ambiente **web**.

Portanto executando o comando `node ace repl` inicializa a aplicação em um ambiente `repl` (Read-Eval-Print Loop). Todos os outros comandos inicializam o app no ambiente do **console**.

O ambiente da aplicação desempenha um papel essencial na decisão de quais ações executar. Por exemplo, o ambiente web não registra ou inicializa os provedores Ace.

Você consegue acessar o ambiente atual da aplicação usando a propriedade `environment`. Segue a lista de ambientes de aplicações conhecidas:

- **web** ambiente referente ao processo inicializado pelo server HTTP.
- **console** ambiente referente aos comandos do Ace execeto pelo comando REPL.
- **repl** ambiente referente ao processo iniciado usando o comando `node ace repl`.
- **test** ambiente reservado para o futuro quando o AdonisJS vai ter imbutido um *test runner*. 
```js
import Application from '@ioc:Adonis/Core/Application'
console.log(Application.environment)
```

## Ciclo de vida de inicialização
Segue um fluxograma do ciclo de vida de inicialização de uma aplicação:
![Boot_Lifecycle_AdonisJS_Image](https://github.com/Greens-Organization/AdonisPT-BR/blob/main/assets/Boot_Lifecycle_AdonisJS.png)
Você pode acessar as ligações do *container* IoC assim que o estado do aplicativo estiver definido como `booted` ou `ready`. Uma tentativa de acessar as ligações do *container* antes do estado inicializado resulta em uma exceção.

Por exemplo, se você tem um provedor de serviço que quer resolver as ligação do *container*, você deve escrever as instruções (statements) de importação dentro dos métodos `boot` ou `ready`.

#### ❌Top-level import will not work
```js
import { ApplicationContract } from '@ioc:Adonis/Core/Application'
import Route from '@ioc:Adonis/Core/Route'

export default class AppProvider {
  constructor(protected app: ApplicationContract) {}

  public async boot() {
    Route.get('/', async () => {})
  }
}
```
#### ✅Move import inside the boot method
```js
import { ApplicationContract } from '@ioc:Adonis/Core/Application'

export default class AppProvider {
  constructor(protected app: ApplicationContract) {}

  public async boot() {
    const { default: Route } = await import('@ioc:Adonis/Core/Route')
    Route.get('/', async () => {})
  }
}
```
## Versão
Você pode acessar a versão da aplicação e do framework usando as propriedades `version` e `adonisVersion`.

A propriedade `version` refere-se a versão dentro do arquivo `package.json` do seu app. A propriedade `adonisVersion` refere-se a versão instalada do pacote `@adonisjs/core`.
```js
import Application from '@ioc:Adonis/Core/Application'

console.log(Application.version!.toString())
console.log(Application.adonisVersion!.toString())
```
Ambas as propriedades de versão são representadas como um objeto com sub-propriedades `major`, `minor`, e `patch`.
```js
console.log(Application.version!.major)
console.log(Application.version!.minor)
console.log(Application.version!.patch)
```
## Ambiente Node
Você pode acessar o ambiente node usando a propriedade `nodeEnvironment`. O valor é uma referência para a variável de ambiente `NODE_ENV`. No entanto, o valor é mais normalizado para ser consistente.
```js
import Application from '@ioc:Adonis/Core/Application'

console.log(Application.nodeEnvironment)
```
| NODE_ENV | Passado para |
| ---------| ------------ |
| dev      | development  |
| develop  | development  |
| stage    | staging      |
| prod     | production   |
| test     | testing      |

Além disso, você pode usar as seguintes propriedades como um atalho para conhecer o ambiente atual.

### inProduction
```js
Application.inProduction

// Igual a
Application.nodeEnvironment === 'production'
```
---
## inDev
```js
Application.inDev

// Igual a
Application.nodeEnvironment === 'development'
```
---
## Crie caminhos para diretórios do projetos
Você pode usar o módulo **Application** para criar um caminho absoluto para diretórios existentes no projeto.

### configPath
Criando um *path* absoluto para um arquivo dentro do diretório `config/`.
```js
Application.configPath('shield.ts')
```
---
### publicPath
Criando um *path* absoluto para um arquivo dentro do diretório `public/`.
```js
Application.publicPath('style.css')
```
---
### databasePath
Criando um *path* absoluto para um arquivo dentro do diretório `database/`.
```js
Application.databasePath('seeders/Database.ts')
```
---
### migrationsPath
Criando um *path* abosluto para um arquivo dentro do diretório `migrations/`.
```js
Application.migrationsPath('users.ts')
```
---
### seedsPath
Criando um *path* absoluto para um arquivo dentro do diretório `seeds/`.
```js
Application.seedsPath('Database.ts')
```
---
### resourcesPath
Criando um *path* absoluto para um arquivo diretório `resources/`.
```js
Application.resourcesPath('scripts/app.js')
```
### viewsPath
Criando um *path* absoluto para um arquivo dentro do diretório `views/`.
```js
Application.viewsPath('welcome.edge')
```
---
### startPath
Criando um *path* absoluto para um arquivo dentro do diretório `start/`.
```js
Application.startPath('routes.ts')
```
---
### tmpPath
Criando um *path* absoluto para um arquivo da aplicação dentro do diretório `tmp/`.
```js
Application.tmpPath('uploads/avatar.png')
```
---
### makePath
Criando um *path* absoludo na raiz da aplicação.
```js
Application.makePath'app/Middleware/Auth.ts')
```
---
## Outras propriedades
A seguir está a lista de propriedades no módulo do aplicativo.

### appName
Nome do aplicativo. Refere-se à propriedade `name` dentro do arquivo `package.json` do seu aplicativo.
```js
Application.appName
```
---
### appRoot
Caminho absoluto para o diretório raiz do aplicativo.
```js
Application.appRoot
```
---
### rcFile
Referência ao arquivo [`.adonisrc.json`](https://docs.adonisjs.com/guides/adonisrc-file).
```js
Application.rcFile.providers
Application.rcFile.raw
```
---
### container
Referência à instância do container IoC.
```js
Application.container
```
---
### helpers
Referência ao módulo *helper*.
```js
Application.helpers.string.snakeCase('helloWorld')
```
Você também pode acessar o módulo de *helpers* diretamente.
```js
import { string } from '@ioc:Adonis/Core/Helpers'

string.snakeCase('helloWorld')
```
---
### logger
Referência ao registrador (**logger**) de aplicativos.
```js
Application.logger.info('hello world')
```
Você também pode acessar o módulo `logger` diretamente.
```js
import Logger from '@ioc:Adonis/Core/Logger'

Logger.info('hello world')
```
---
### config
Referência ao módulo de configuração.
```js
Application.config.get('app.secret')
```
Você também pode acessar o módulo `config` diretamente.
```js
import Config from '@ioc:Adonis/Core/Config'

Config.get('app.secret')
```
---
### env
Referência ao módulo de secretos/chaves.
```js
Application.env.get('APP_KEY')
```
Você também pode acessar o módulo `env` diretamente.
```js
import Env from '@ioc:Adonis/Core/Env'

Env.get('APP_KEY')
```
---
### isReady
Descobre se a aplicação está no `state: ready`. É usado internamente para parar de aceitar novas solicitações HTTP quando `isReady` for `false`.
```js
Application.isReady
```
---
### isShuttingDown
Descobre se a aplicação está no processo de desligamento.
```js
Application.isShuttingDown
```
<footer align="center">
  <a href="./02-AdonisRC_file.md">Ir para AdonisRC file</a>
</footer>
