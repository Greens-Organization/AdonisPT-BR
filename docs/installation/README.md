# Instalação
AdonisJS é uma estrutura `Node.js` e, portanto, requer que o Node.js esteja instalado em seu computador. Para ser preciso, precisamos de pelo menos a versão mais recente do `Node.js v14`.

Você pode verificar as versões do [Node](https://nodejs.org/en/) e [npm](https://www.npmjs.com) executando os comandos a seguir.
```
# checa a versão do node.js
node -v
```
Se você não tiver o Node.js instalado, você pode [baixá-lo](https://nodejs.org/en/download/) para o seu sistema operacional no site oficial.

Se você estiver familiarizado com a linha de comando, recomendamos você usar [Volta](https://volta.sh) ou o [Node Version Manager](https://github.com/nvm-sh/nvm), ou até mesmo o [asdf](https://asdf-vm.com) com o [plugin do Node.js], para instalar e executar várias versões do Node.js em seu computador.

## Criando um novo projeto
Você pode criar um novo projeto usando [npm init](https://docs.npmjs.com/cli/v7/commands/npm-init), [yarn create](https://classic.yarnpkg.com/en/docs/cli/create) ou [pnpm create](https://pnpm.io/tr/next/cli/create). Essas ferramentas baixarão o pacote inicial do AdonisJS e iniciarão o processo de instalação.
- Para npm:
```
npm init adonis-ts-app@latest hello-world
```
- Para o yarn:
```
yarn create adonis-ts-app hello-world
```
- Para o pnpm:
```
pnpm create adonis-ts-app hello-world
```
O processo de instalação vai lhe pedir as seguintes seções.

### Estrutura do projeto
Você pode escolher entre uma das seguintes estruturas de projeto.

- `web` é ideal para criar aplicações clássicas renderizados pelo servidor. Configuramos o suporte para sessões e também instalamos o template engine AdonisJS.
- `api` a estrutura do projeto é ideal para criar um servidor de API.
- `slim` a estrutura do projeto cria o menor aplicativo AdonisJS possível e não instala nenhum pacote adicional, execto o [núcleo](https://github.com/adonisjs/core) do framework.

### Nome do projeto
o nome projeto. Definimos o valor desse prompt dentro do arquivo `package.json`.

### Configurar o eslint/prettier
Opcionalmente, você pode configurar o eslint e o prettier. Ambos os pacotes são configurações criados de forma seletiva pela equipe principal do AdonisJS.

### Configurar o Webpack Encore
Opcionalmente, você também pode configurar o [Webpack Encore](https://docs.adonisjs.com/guides/assets-manager) para agrupar e servir dependências de front-end.

Observe que o AdonisJS é uma estrutura de back-end e não se preocupa com ferramentas de construção de front-end. Portanto, a configuração do Webpack é opcional.

## Iniciando o servidor de desenvolvimento
Depois de criar o aplicativo, você pode iniciar o servidor de desenvolvimento executando o comando a seguir.
```
node ace serve --watch
```
- O comando `serve` inicia o servidor HTTP e executa uma compilação na memória do TypeScript para JavaScript.
- A flag `--watch` destina-se a observar o sistema de arquivos em busca de alterações e reinicia o servidor automaticamente.

Por padrão, o servidor inicia na porta 3333 (definida dentro do arquivo `.env`). Você pode visualizar a página de boas-vindas visitando: http://localhost:3333

## Compilando para produção
Você deve sempre dá deploy do seu JavaScript já compilado em seu seu servidor de produção. Você pode criar a compilação de produção executando o seguinte comando:
```
node ace build --production
```
A saída compilada é gravada na pasta `build/`. Você pode fazer `cd` nesta pasta e iniciar o servidor executando diretamente o arquivo `server.js`. Saiba mais sobre o [processo de compilação do TypeScript](https://docs.adonisjs.com/guides/typescript-build-process).
```
cd build
node server.js
```