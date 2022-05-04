# Config
A Config do tempo de execução do seu aplicativo AdonisJS é armazenada dentro do diretório de configuração. O núcleo da estrutura e muitos dos pacotes instalados dependem desse arquivo de configuração. Portanto, verifique os arquivos de configuração e ajuste as configurações (se necessário).

Também recomendamos armazenar todas as configurações personalizadas exigidas pelo seu aplicativo dentro desse diretório em vez de armazená-las em vários locais.

## Importando arquivos de configurações
Você pode importar os arquivos configurações dentro do seu código do app usando a instrução `import`. Por exemplo:
```js
import { appKey } from 'Config/app'
```
## Usando o provedor de configuração
Em vez de importar diretamente os arquivos de configuração, você também pode usar o provedor de `Config` da seguinte maneira:
```ts
import Config from '@ioc:Adonis/Core/Config'

Config.get('app.appkey')
```
O método `Config.get` aceita um caminho separado por ponto (`app.`) para a o `appKey`. No exemplo acima, lemos a propriedade `appKey` do arquivo `config/app.ts`.

Além disso, você pode definir um valor de fallback. O valor de fallback é retornado quando o valor de configuração real está ausente.
```js
Config.get('database.connections.mysql.host', '120.0.0.1')
```
Não há benefícios diretos de usar o **Config provider** sobre a importação manual dos arquivos de configuração. No entanto, o **Config provider** é a única opção nos cenários a seguir.

- **Pacotes externos**: nunca deve confiar no caminho dos arquivos dos pacotes externos para ler/importar a configuração. Em vez disso, deve ser usado o **Config provider**. O uso do provedor **Config** cria uma ligação fraca entre o aplicativo e o pacote.
- **Modelos de borda**: os arquivos de template podem usar o método global [config](https://docs.adonisjs.com/reference/views/globals/all-helpers#config) para fazer referência aos valores de configuração.

## Mudando locais de configuração
Você pode atualizar o local do diretório de configuração modificando o arquivo `.adonisrc.json`.
```json
"directories": {
  "config": "./configurations"
}
```
O provedor de configuração vai ler automaticamente o arquivo do diretório recém-configurado e todos os pacotes subjacentes que dependem dos arquivos de configuração funcionarão bem.

## Algumas ressalvas
Todos os arquivos de configuração dentro do diretório de configuração são importados automaticamente pela estrutura durante a fase de inicialização. Como resultado disso, seus arquivos de configuração não devem depender das ligações do container.

Por exemplo, o código a seguir será interrompido ao tentar importar o provedor antes de ser registrado no container.
```js
// ❌ não funciona
import Route from '@ioc:Adonis/Core/Route'

const someConfig = {
  assertsUrl: Route.makeUrl('/assets')
}
```
Você pode considerar essa limitação ruim. No entanto, tem um impacto positivo no design do aplicativo.

Fundamentalmente, seu código de tempo de execução deve depender da configuração e NÃO o contrário. Por exemplo:

❌ Não faz isso
```js
import User from 'App/Models/User'

const someConfig = {
  databaseTable: User.table
}
```
✅ Em vez disso faz isso
```js
const someConfig = {
  databaseTable: 'users'
}
```
```js
// em outro arquivo
import someConfig from 'Config/file/path'

class User extends Model {
  public static table = someConfig.databaseTable
}
```
## Referências de configuração
Conforme você instala e configura os pacotes do AdonisJS, eles podem criar novos arquivos de configuração. A seguir está uma lista de arquivos de configuração (com seus modelos padrão) usados pelas diferentes partes da estrutura.

| Arquivo Config | Esboço | Usado pelo |
| -------------- | ------ | ---------- |
| `app.ts`       | https://git.io/JfefZ | Usado pelo núcleo da estrutura, incluindo o servidor HTTP, registrador, validador e gerenciador de assets. |
| `bodyparser.ts` | https://git.io/Jfefn | Usado pelo middleware bodyparser |
| `cors.ts` | https://git.io/JfefC | Usado pelo gancho do servidor CORS |
| `hash.ts` | https://git.io/JfefW | Usado pelo pacote hash |
| `session.ts` | https://git.io/JeYHp | Usado pelo pacote de sessão |
| `shield.ts` | https://git.io/Jvwvt | Usado pelo pacote de guard |
| `static.ts` | https://git.io/Jfefl | Usado pelo servidor de arquivos estáticos |
| `auth.ts` | https://git.io/JY0mp | Usaado pelo pacote de autenticação |
| `database.ts` | https://git.io/JesV9 | Usado pelo Lucid ORM |
| `mail.ts` | https://git.io/JvgAf | Usado pelo pacote de AdonisJS mail |
| `redis.ts` | https://git.io/JemcF | Usado pelo pacote do Redis |
| `drive.ts` | https://git.io/JBt3o | Usado pelo provedor do Drive |
| `ally.ts` | https://git.io/JOdi5 | Usado pelo pacote de autenticação Social (Ally) |