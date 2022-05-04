# Variaveis de ambiente 
Em vez de manter vários arquivos de configuração, um para cada ambiente, o AdonisJS usa [variáveis de ambiente](https://en.wikipedia.org/wiki/Environment_variable) para valores que geralmente mudam entre o ambiente local e o de produção. Por exemplo: as credenciais do banco de dados, um sinaliador booleano para alternar o cache de modelos e assim por diante.

## Acessar variáveis de ambiente
O Node.js permite nativamente acessar as variáveis de ambiente usando o objeto `process.env`. Por exemplo:
```
process.env.NODE_ENV
process.env.HOST
process.env.PORT
```
No entanto, recomendamos o uso do provedor **AdonisJS Env**, pois ele aprimora ainda mais a API para trabalhar com variáveis de ambiente, adicionando suporte para validações e fornecendo informações de tipo estático.
```js
import Env from '@ioc:Adonis/Core/Env'

Env.get('NODE_ENV')

// Com valores padrões
Env.get('HOST', '0.0.0.0')
Env.get('PORT', 3333)
```
## Porque validar variáveis de ambiente?
As variáveis de ambiente são injetadas de fora para dentro em seu app e você tem pouco ou nenhum controle sobre elas dentro do seu código.

Por exemplo, uma seção de seu código depende da existência de uma variável de ambiente `SESSION_DRIVER`.
```js
const driver = process.env.SESSION_DRIVER

// Código fictício
await Session.use(driver).read()
```
Não há garantia de que no momento da execução do programa, a variável `SESSION_DRIVER` exista e tenha o valor correto. Portanto, você deve validá-lo em vez de obter um erro mais tarde no ciclo de vida do programa reclamando de algo "deu erro aqui".
```js
const driver = process.env.SESSION_DRIVER

if(!driver) {
  throw new Error('Missing env variable "SESSION_DRIVER"')
}

if (!['memory', 'file', 'redis'].includes(driver)) {
  throw new Error('Invalid value for env variable "SESSION_DRIVER"')
}
```
Agora imagine escrever essas condicionais em todo local do seu código? **Bem, não mostra uma grande experiência com desenvolvimento, sorry!**

## Validando variáveis de ambiente
O AdonisJS permite que você valide opcionalmente as variáveis de ambiente muito cedo no ciclo de vida de inicialização do seu aplicativo e se recusa a iniciar se alguma validação falhar.

Você começa definindo as regras de validação dentro do arquivo `env.ts`.
```js
import Env from '@ioc:Adonis/Core/Env'

export default Env.rules({
  HOST: Env.schema.string({ format: 'host' }),
  PORT: Env.schema.number(),
  APP_KEY: Env.schema.string(),
  APP_NAME: Env.schema.string(),
  CACHE_VIEWS: Env.schema.boolean(),
  SESSION_DRIVER: Env.schema.string(),
  NODE_ENV: Env.schema.enum(['development', 'production', 'testing'] as const),
})
```
Além disso, o AdonisJS extrai as informações de tipo estático das regras de validação e fornece o IntelliSense para as propriedades validadas.
![IntelliSense_Image](../assets/Usando_IntelliSense.webp)
## Schema API
A seguir está a lista de métodos disponíveis para validar as variáveis de ambiente.

### Env.schema.string
Válida que o valor exista e seja uma string válida. Strings vazias falham nas validações e você deve usar a variante opcional para permitir strings vazias.
```js
{
  APP_KEY: Env.schema.string()
}

// Marcando como opcional
{
  APP_KEY: Env.schema.string.optional()
}
```
Você também pode forçar o valor a ter um dos formatos predefinidos.
```js
// Deve ser um host válido (url ou ip)
Env.schema.string({ format: 'host' })

// Deve ser uma URL válida
Env.schema.string({ format: 'url' })

// Deve ser um endereço de e-mail válido
Env.schema.string({ format: 'email' })
```
Ao validar o formato de `url`, você também pode definir opções adicionais para forçar/ignorar o `tld` e o `protocolo`.
```js
Env.schema.string({ format: 'url', tld: false, protocol: false })
```
### Env.schema.boolean
Força o valor a ser uma representação de string válida de um booleano. Os valores a seguir são considerados booleanos válidos e serão convertidos em `true` ou `false`.

- `'1', 'true'` são lançados para `Boolean(true)`
- `'0', 'false'` são lançados para `Boolean(false)`
```js
{
  CACHE_VIEWS: Env.schema.boolean()
}

// Marcando como opcional
{
  CACHE_VIEWS: Env.schema.boolean.optional()
}
```
### Env.schema.number
FOrça o valor a ser uma representação de string válida de um número.
```js
{
  PORT: Env.schema.number()
}

// Marcando como opcional
{
  PORT: Env.schema.number.optional()
}
```
### Env.schema.enum
Força o valor a ser um dos valores predefinidos.
```js
{
  NODE_ENV: Env
    .schema
    .enum(['development', 'production'] as const)
}

// Marcando como opcional
{
  NODE_ENV: Env
    .schema
    .enum
    .optional(['development', 'production'] as const)
}
```
### Funções personaliadas
Para todos os outros casos de uso de validação, você pode definir suas funções personalizadas.
```js
{
  PORT: (key, value) => {
    if (!value) {
      throw new Error('Value for PORT is required')
    }
    
    if (isNaN(Number(value))) {
      throw new Error('Value for PORT must be a valid number')    
    }

    return Number(value)
  }
}
```
- Certifque-se de sempre retornar o valor após validá-lo.
- O valor de retorno pode ser diferente do valor de entrada inical.
- Interimos o tipo estático do valor de retorno. Neste caso, `Env.get('PORT')` é um número.

### Definindo variáveis no desenvolvimento
Durante o desenvolvimento, você pode definir variáveis de ambiente dentro do arquivo `.env` armazenado na raiz do seu projeto, e o AdonisJS o processará automaticamente.
```
// .env
PORT=3333
HOST=0.0.0.0
NODE_ENV=development
APP_KEY=sH2k88gojcp3PdAJiGDxof54kjtTXa3g
SESSION_DRIVER=cookie
CACHE_VIEWS=false
```
### Substituindo variáveis
Junto com o suporte padrão para analisar o arquivo `.env`, o AdonisJS também permite substituição de variáveis.
```
HOST=localhost
PORT=3333
URL=$HOST:$PORT
# Mesmo que URL=localhost:3333
```
Todas as `letter`, números, e sublinhados (`_`) após o cifrão (`$`) são analisados como variáveis. Se sua variável contém qualquer outro caractere, você deverá envolvê-lo dentro de chaves, {}.
```
REDIS-USER=foo
REDIS-URL=localhost@${REDIS-USER}
```
### Fuja do sinal $
Se o valor de uma variável contém um sinal `$`, você deve escapar (com `\`) dele para evitar a substituição da variável.
```
PASSWORD=pa\$\$word
```
### Não dê commit no arquivo .env
O arquivo `.env` não são portáteis. Ou seja, as credencias do banco de dados em seu ambiente local e de produção sempre serão diferentes e, portanto, não faz sentido enviar o `.env` para o controle de versionamento.

Você deve considerar o arquivo `.env` pessoal para seu ambiente local e criar um arquivo `.env` separado na produção ou no servidor de teste (e mantê-lo seguro).
O arquivo `.env` pode estar em qualquer local em seu servidor. Por exemplo, você pode armazená-lo dentro de `/etc/myapp/.env` e então informar o AdonisJS sobre ele da seguinte forma.
```
ENV_PATH=/etc/myapp/.env node server.js
```
## Definindo variáveis durante o teste
O AdonisJS procurará o arquivo `.env.testing` quando o aplicativo for iniciado com a variável de ambiente `NODE_ENV=testing`.

As variáveis definidas dentro do arquivo `.env.testing` são autoticamente mesclados com o arquivo `.env`. Isso permite que você use um banco de dados diferente ou um driver de sessão diferente ao escrever testes.

## Definindo variáveis em produção
A maioria dos provedores de hospadagem modernos tem suporte de primeira classe para definir variáveis de ambiente em seu console da web. Certifique-se de ler a documentação do seu provedor de hospadagem e definir as variáveis de ambiente antes de implantar seu aplicativo AdonisJS.

<footer align="center">
  <a href="./03-Config.md">Voltar para Config ------------------------</a>
  <a href="./05-TypeScript_build_process.md">Ir para TypeScript Build Process</a>
</footer>