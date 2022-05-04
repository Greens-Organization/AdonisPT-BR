# Arquivo AdonisRC
O arquivo `.adonisrc.json` é armazenado na raiz do seu projeto. Ele configura o espaço de trabalho e algumas das configurações de tempo de execução do seu aplicativo AdonisJS.

O arquivo contém apenas a configuração mínima necessária para executar seu aplicativo. No entanto, você pode visualizar o conteúdo completo do arquivo executando o seguinte comando Ace.
```
node ace dump:rcfile
```
```json
// Output do comando acima
{
  "typescript": true,
  "directories": {
    "config": "config",
    "public": "public",
    "contracts": "contracts",
    "providers": "providers",
    "database": "database",
    "migrations": "database/migrations",
    "seeds": "database/seeders",
    "resources": "resources",
    "views": "resources/views",
    "start": "start",
    "tmp": "tmp",
    "tests": "tests"
  },
  "exceptionHandlerNamespace": "App/Exceptions/Handler",
  "preloads": [
    {
      "file": "./start/routes",
      "optional": false,
      "environment": [
        "web",
        "console",
        "test"
      ]
    },
    {
      "file": "./start/kernel",
      "optional": false,
      "environment": [
        "web",
        "console",
        "test"
      ]
    },
    {
      "file": "./start/views",
      "optional": false,
      "environment": [
        "web",
        "console",
        "test"
      ]
    },
    {
      "file": "./start/events",
      "optional": false,
      "environment": [
        "web"
      ]
    }
  ],
  "namespaces": {
    "models": "App/Models",
    "middleware": "App/Middleware",
    "exceptions": "App/Exceptions",
    "validators": "App/Validators",
    "httpControllers": "App/Controllers/Http",
    "eventListeners": "App/Listeners",
    "redisListeners": "App/Listeners"
  },
  "aliases": {
    "App": "app",
    "Config": "config",
    "Database": "database",
    "Contracts": "contracts"
  },
  "metaFiles": [
    {
      "pattern": "public/**",
      "reloadServer": false
    },
    {
      "pattern": "resources/views/**/*.edge",
      "reloadServer": false
    }
  ],
  "commands": [
    "./commands",
    "@adonisjs/core/build/commands",
    "@adonisjs/repl/build/commands"
  ],
  "commandsAliases": {
  },
  "providers": [
    "./providers/AppProvider",
    "@adonisjs/core",
    "@adonisjs/session",
    "@adonisjs/view"
  ],
  "aceProviders": [
    "@adonisjs/repl"
  ]
}
```
### typescript
A propriedade `typescript` informa ao framework e aos comandos Ace que sua aplicação está usando TypeScript. Atualmente, esse valor é sempre definido como `true`. No entanto, mais tarde permitiremos que os aplicativos também sejam escritos em JavaScript.

### directories
Um objeto de diretórios conhecidos e seus caminhos pré-configurados. Você pode alterar o caminho para corresponder aos requisitos.

Além disso, todos os comandos do Ace fazem referência ao arquivo `.adonisrc.json` antes de criar o arquivo.
```json
{
  "directories": {
    "config": "config",
    "public": "public",
    "contracts": "contracts",
    "providers": "providers",
    "database": "database",
    "migrations": "database/migrations",
    "seeds": "database/seeders",
    "resources": "resources",
    "views": "resources/views",
    "start": "start",
    "tmp": "tmp",
    "tests": "tests"
  }
}
```
### exceptionHandlerNamespace
O namespace para a classe que manipula exceções ocorre durante uma solicitação HTTP.
```json
{
  "exceptionHandlerNamespace": "App/Exceptions/Handler"
}
```
### preloads
Um array de arquivos a serem carregados no momento da inicialização do aplicativo. Os arquivos são carregados logo após a inicialização dos provedores de serviços.

Você pode definir o ambiente no qual carregar o arquivo. As opções válidas são:

- `web` ambiente referente ao processo iniciado para o servidor HTTP.
- `console` ambiente referente aos comandos Ace, exceto pelo comando `repl`.
- `repl` ambiente referente ao processo iniciado usando o comando `node ace repl`.
- Finalmente, ambiente `test` é reservado para no futuro quando o AdonisJS tiver imbutido um *test runner*.

Além disso, você pode marcar o arquivo como opcional e o ignoraremos se o arquivo estiver ausente no disco.

> Você pode criar e registrar um arquivo pré-carregado executando o comando `node ace make:prldfile`.

```json
{
  "preloads": [
    {
      "file": "./start/routes",
      "optional": false,
      "environment": [
        "web",
        "console",
        "test"
      ]
    },
  ]
}
```
### namespace
Um objeto de namespace para as entidades conhecidas.

Por exemplo, você pode alterar o namespace do controlador de `App/Controllers/Http` para `App/Controllers` e manter os controladores dentro do diretório `./app/Controllers`.
```json
{
  "namespaces": {
    "controllers": "App/Controllers"
  }
}
```
### aliases
A propriedade `aliases` permite definir os *aliases* de importação para diretórios específicos. Após definir o alias, você poderá importar arquivos da raiz do diretório de *aliases*.

No exemplo a seguir, o App é um `alias` para o diretório `./app` e o restante é o caminho do arquivo do diretório fornecido.
```js
import 'App/Models/User'
```
Os aliases do AdonisJS são apenas para tempo e execução. Você também terá que registrar o mesmo alias dentro do arquivo `tsconfig.json` para que o compilador TypeScript funcione.
```json
{
  "compilerOptions": {
    "paths": {
      "App/*": [
        "./app/*"
      ],
    }
  }
}
```
### metaFiles
O array `metaFiles` aceita os arquivos que você deseja que o AdonisJS copie para a pasta de compilação ao criar a compilação de produção.

Você pode definir os caminhos de arquivo como um **pattern glob** e copiaremos todos os arquivos correspondentes para esse *pattern*. Você também pode instruir o servidor de desenvolvimento a recarregar quaisquer arquivos dentro das alterações de padrão correspondentes.
```json
{
  "metaFiles": [
    {
      "pattern": "public/**",
      "reloadServer": false
    },
    {
      "pattern": "resources/views/**/*.edge",
      "reloadServer": false
    }
  ]
}
```
### commands**
Um array de caminhos para comandos Ace de `lookup/index`. Você pode definir um caminho relativo como `./command` ou caminho para um pacote instalado.
```json
{
  "commands": [
    "./commands",
    "@adonisjs/core/build/commands"
  ]
}
```
### commandsAliases
Um par chave-valor de aliases de comando. Isso geralmente é para ajudá-lo a criar **memorable aliases** para os comandos que são mais difíceis de digitar ou lembrar.
```json
{
  "migrate": "migration:run"
}
```
Você também pode definir múltiplos aliases.
```json
{
  "migrate": "migration:run",
  "up": "migration:run"
}
```
### providers
Um array de **providers** de serviços para carregar durante o ciclo de inicialização do aplicativo. Os provedores mencionados neste array são carregados em todos os ambientes.
```json
{
  "providers": [
    "./providers/AppProvider",
    "@adonisjs/core"
  ],
}
```
### aceProviders
Um array de **providers** para carregar ao executar os comandos Ace. Aqui você pode carregar os **providers**, que não são necessários ao iniciar o servidor HTTP.
```json
{
  "aceProviders": [
    "@adonisjs/repl"
  ]
}
```