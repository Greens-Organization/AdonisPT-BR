# Deployment
O deploy de um aplicativo AdonisJS não é diferente do deploy de qualquer outro aplicativo Node.js. Primeiro, você precisará de um servidor/plataforma que possa instalar e executar a versão mais recente do `Node.js v14`.

> Para uma experiência de deployment sem atritos, você pode experimentar o Cleavr. É um serviço de provisionamento de servidor e possui suporte de primeira para o [deploy de aplicações AdonisJS](https://cleavr.io/adonis/).
> 
> Disclaimer: Cleavr também é patrocinador do AdonisJS

## Compilando do TypeScript para JavaScript
Os aplicativos AdonisJS são escritos em TypeScript e devem ser compilados para JavaScript durante o deployment. Você pode compilar seu aplicativo diretamente no servidor de produção ou executar a etapa de compilação em um pipeline de CI/CD.

Você pode dar build no seu código para produção executando o seguinte comando Ace. A saída JavaScript compilada é gravada no diretório `build`.
```
node ace build --production
```
Se você executou a etapa de compilação dentro de um pipeline de CI/CD, poderá mover apenas a pasta `build` para o servidor de produção e instalar as dependências de produção diretamente no servidor.

## Iniciando o servidor de produção
Você pode iniciar o servidor de produção executando o arquivo `server.js`.

Se você executou a etapa de build em seu servidor de produção, certifique-se de primeiro dá `cd build` e, em seguida, inicie o servidor.
```
cd build
npc ci --production

# Iniciando o servidor
node server.js
```
Se a etapa de compilação foi executada em um pipeline de CI/CD e você copiou apenas a pasta `build` para seu servidor de produção, a pasta `build` se tornará a raiz do seu app.
```
npmc ci --production

# Iniciando o servidor
node server.js
```
### Usando um gerenciador de processos
Recomenda-se usar um gerenciador de processos ao gerenciar um aplicativo Node.js em um servidor básico.

Um gerenciador de processos garante a reinicialização do aplicativo se ele travar durante o tempo de execução. Além disso, alguns gerenciadores de processos, como o [PM2](https://pm2.keymetrics.io/docs/usage/quick-start/), também podem executar reinicializações normais ao reimplementar (re-deploying) o app.

A seguir está um [arquivo do ecossistema](https://pm2.keymetrics.io/docs/usage/application-declaration/) de exemplo para PM2.
```js
module.exports = {
  apps: [
    {
      name: 'web-app',
      script: './build/server.js',
      instances: 'max',
      exec_mode: 'cluster',
      autorestart: true,
    },
  ],
}
```
## Proxy reverso NGINX
Ao executar o aplicativo AdonisJS em um servidor **bare-bone**, você deve colocá-lo atrás do NGINX (ou em um servidor da Web semelhante) por [vários motivos diferentes](https://medium.com/intrinsic-blog/why-should-i-use-a-reverse-proxy-if-node-js-is-production-ready-5a079408b2ca), mas a terminação SSL é importante.

> Certifique-se de ler [o guia de proxies confiáveis](https://docs.adonisjs.com/guides/request#trusted-proxy) que você possa acessar o endereço IP correto ao visitante ao executar o aplicativo AdonisJS atrás de um servidor proxy.

A seguir está um exemplo de configuração NGINX para solicitações de proxy para seu aplicativo AdonisJS. **Certifique-se de substituir os valores dentro dos colchetes angulares** `<>`.
```
server {
  listen 80;

  server_name <APP_DOMAIN.COM>;

  location / {
    proxy_pass http://localhost:<ADONIS_PORT>;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_cache_bypass $http_upgrade;
  }
}
```
## Migrando banco de dados
Usando o comando `node ace migration:run --force`, você pode migrar seu banco de dados de produção. A flag `--force` é necessária ao executar migrações no ambiente de produção.

### Quando migrar
Além disso, seria melhor se você sempre executasse as migrações antes de reiniciar o servidor. Então, se a migração falhar, não reinicie o servidor.

Usando um serviço gerenciado como Clearv ou Heroku, eles podem lidar automaticamente com esse caso de uso. Caso contrário, você terá que executar o script de migração dentro de um pipeline de CI/CD ou executá-lo manualmente por meio de SSH.

### Não faça rollback na produção
O método `down` em seus arquivos de migração geralmente contém ações destrutivas como descartar a tabela ou remover uma coluna e assim por diante. Recomenda-se desativar os rollbacks em produção dentro do arquivo `config/database.ts`.

Desabilitar os rollbacks na produção garantirá que o comando `node ace migration:rollback` resulte em um erro.
```js
{
  pg: {
    client: 'pg',
    migrations: {
      disableRollbacksInProduction: true,
    }
  }
}
```
### Evite tarefas de migração simultâneas
Ao dá deploy no seu aplicativo AdonisJS em vários servidores, certifique-se de executar as migrações de apenas um servidor e não de todos eles.

Para MySQL e PostgreSQL, o Lucid obterá [bloqueios consultivos](https://www.postgresql.org/docs/9.4/explicit-locking.html#ADVISORY-LOCKS) para garantir que a migração simultânea não seja permitida. No entanto, é melhor evitar executar migrações de vários servidores em primeiro lugar.

## Armazenamento persistente para uploads de arquivos
Plataformas de deployment modernas, como Amazon ECS, Heroku ou DigitalOcean, executam o código do aplicativo dentro de um [sistema de arquivo efêmero](https://devcenter.heroku.com/articles/dynos#ephemeral-filesystem), o que significa que cada deployment destruirá o sistema de arquivos existente e criará um novo.

Você perderá os arquivos carregados pelo usuário se armazenados no mesmo armazenamento que o código do aplicativo. Portanto, você deve considerar o uso do [Drive](https://docs.adonisjs.com/guides/drive) para manter os arquivos carregados pelo usuário em um serviço de armazenamento em nuvem, como Amazon S3 ou Google Cloud Storage.

## Logging
O [AdonisJS Logger](https://docs.adonisjs.com/guides/logger) grava logs em `stdout` e `stderr` no formato JSON. Você pode configurar um serviço de log externo para ler os logs de `stdout/stderr` ou encaminhá-los para um arquivo local no mesmo servidor.

O núcleo da estrutura e os pacotes do ecossistema gravam logs no nível de `trace`. Portanto, você deve definir o nível do log para **rastrear** (`trace`) ao depurar os componentes internos da estrutura.

## Debugando consultas de banco de dados
O Lucid ORM emite o evento `db:query` quando a depuração do banco de dados está ativada. Você pode ouvir este evento e debugar as consultas SQL usando o Logger.

A seguir está um exemplo de impressão (bonita, diga-se de passagem) das consultas de banco de dados em desenvolvimento e uso do Logger em produção.
```js
// start/events.ts
import Event from '@ioc:Adonis/Core/Event'
import Logger from '@ioc:Adonis/Core/Logger'
import Database from '@ioc:Adonis/Lucid/Database'
import Application from '@ioc:Adonis/Core/Application'

Event.on('db:query', (query) => {
  if (Application.inProduction) {
    Logger.debug(query)
  } else {
    Database.prettyPrint(query)
  }
})
```

## Variáveis de Ambiente
Você deve manter suas variáveis de ambiente de produção seguras e não as mantenha junto com o código do aplicativo. Se você estiver usando uma plataforma de deployment como Cleavr, Heroku e assim por diante, deverá gerenciar as variáveis de ambiente a partir do painel web.

Quando você dá deploy do seu código em um servidor básico, você pode manter suas variáveis de ambiente dentro do arquivo `.env`. O arquivo também pode ficar fora da base do código do aplicativo. Certifique-se de informar ao AdonisJS sobre sua localização usando a variável de ambiente `ENV_PATH`.
```
cd build

ENV_PATH=/etc/myapp/.env node server.js
```
## Cache de Views
Você deve armazenar em cache os modelos do Edge em produção usando a variável de ambiente `CACHE_VIEWS`. Os modelos são armazenados em cache na memória em tempo de execução e nenhuma pré-compilação é necessária.
```
CACHE_VIEWS=true
```
## Como veicular ativos estáticos
Servir assets estáticos de forma eficaz é essencial para o desempenho de seu aplicativo. Independemente da velocidade de seus aplicativos AdonisJS, a entrega de assets estáticos desempenha um papel importante em uma melhor experiência do usuário.

### Usando um serviço CDN
A melhor abordagem é usar um CDN para entregar os assets estáticos do seu aplicativo AdonisJS.

Os assets de front-end compilados usando [Webpack Encore](https://docs.adonisjs.com/guides/assets-manager) têm impressão digital, e isso permite que seu servidor CDN os armazene em cache de forma agressiva.

Dependendo do serviço CDN que você está usando e sua técnica de deployment, talvez seja necessário adicionar uma etapa no processo de deployment para mover os arquivos estáticos para o servidor CDN. É assim que deve funcionar em poucas palavras.

- Atualize o `webpack.config.js` para uasar a URL da CDN quando criar o build de produção.
```js
if (Encore.isProduction()) {
  Encore.setPublicPath('https://your-cdn-server-url/assets')
  Encore.setManifestKeyPrefix('assets/')
} else {
  Encore.setPublicPath('/assets')
}
```
- Construa seu aplicativo AdonisJS como de costume. 
- Copie a saída de `public/assets` para seu servidor CDN. Por exemplo, [aqui está um comando](https://github.com/adonisjs-community/polls-app/blob/main/commands/PublishAssets.ts) que usamos para publicar os assets em um bucket do Amazon S3.

### Usando NGINX para entregar arquivos estáticos
Outra opção é descarregar a tarefa de servidor assets para o NGINX. Se você usar o Webpack Encore para compilar os recursos do front-end, você deve armazenar em cache agressivamente todos os arquivos estáticos, pois eles têm impressão digital.

Adicione o seguinte bloco ao seu arquivo de configuração NGINX. Certifique-se de substituir os valores dentro dos colchetes angulares `<>`.
```
location ~ \.(jpg|png|css|js|gif|ico|woff|woff2) {
  root <PATH_TO_ADONISJS_APP_PUBLIC_DIRECTORY>;
  sendfile on;
  sendfile_max_chunk 2mb;
  add_header Cache-Control "public";
  expires 365d;
}
```
### Usando o servidor de arquivos estáticos no AdonisJS
Você também pode contar com o servidor de arquivos estáticos embutido do AdonisJS para servir os assets estáticos do diretório `public/` para manter as coisas simples.

Nenhuma configuração adicional é necessária. Basta dá deploy na sua aplicação AdonisJS como de costume, e a solicitação de assets estáticos será atendida automaticamente. No entanto, se você usar o Webpack Encore para compilar seus recursos do front-end, deverá atualizar o arquivo `config/static.ts` com as seguintes opções.
```js
// config/static.ts
{
  // ... resto da config
  maxAge: '365d',
  immutable: true,
}
```