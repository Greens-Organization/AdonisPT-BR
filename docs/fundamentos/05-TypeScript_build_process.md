# Processo de compilação do TypeScript
Um dos objetivos do framework é fornecer suporte de primeira classe para TypeScript. Isso vai além dos tipos estáticos e do IntelliSense que você pode aproveitar enquanto escreve o código.

Também garantimos que você nunca precisará instalar nenhuma ferramenta de compilação adicional para compilar seu código durante o desenvolvimento ou para a produção.

> Esse guia assume que você tem algum conhecimento sobre TypeScript e o ecossistema de ferramentas de compilação.

## Abordagens comuns de agrupamento
A seguir estão algumas das abordagens comuns para desenvolver um aplicativo Node.js escrito em TypeScript.

### Usando o tsc
A maneira mais simples de compilar seu código TypeScript para JavaScript é usando a linha de comando oficial tsc.

- Durante o desenvolvimento, você pode compilar seu código no modo watch usando o comando tsc --watch. 
- Em seguida, você pode pegar o `nodemon` para observar a saída compilada (código JavaScript) e reiniciar o servidor HTTP a cada alteração. A essa altura, você tem dois observadores em execução. 
- Além disso, talvez seja necessário escrever [scripts personalizados para copiar arquivos estáticos](https://github.com/microsoft/TypeScript/issues/30835), como modelos, para a pasta de compilação, para que seu código JavaScript de tempo de execução possa localizá-lo e referenciá-lo.

### Usando o ts-node
O ts-node melhora a experiência de desenvolvimento, pois compila o código na memória e não o gera no disco. Assim, você pode combinar `ts-node` e `nodemon` e executar seu código TypeScript como um cidadão de primeira classe.

No entanto, para aplicativos maiores, o `ts-node` pode ficar lento, pois precisa recompilar todo o projeto a cada a cada alteração de arquivo. Em contrastem o tsc reconstruindo apenas o arquivo alterado.

Observe que o `ts-node` é uma ferramenta somente de desenvolvimento. Portanto, você ainda precisa compilar seu código para JavaScript usando `tsc` e escrever scripts personalizados para copiar arquivos estáticos para produção.

### Usando o webpack
Depois de tentar as abordagens acima, você pode decidir experimentar o Webpack. Webpack é uma ferramenta de construção e tem muito a oferecer. Mas, ele vem com seu próprio conjunto de desvantagens.

- Em primeiro lugar, usar o Webpack para agrupar o código de back-end é um exagero. Talvez você nem precise de 90% dos recursos do Webpack criados para atender ao ecossistema de front-end.
- Você pode ter que repetir algumas das configurações no arquivo `webpack.config.js` e `tsconfig.json` principalmente, quais arquivos observar e ignorar.
- Além disso, não temos certeza se você pode instruir o [Webpack A NÃO agrupar](https://stackoverflow.com/questions/40096470/get-webpack-not-to-bundle-files) todo o back-end em um único arquivo.

## Abordagens do AdonisJS
Não somos um grande fã de ferramentas de construção complicadas e compiladores de ponta. Ter uma experiência de desenvolvimento calma é muito mais valioso do que expor a configuração para ajustar cada botão.

Começamos com o seguinte conjunto de metas.

- Atenha-se ao compilador oficial do TypeScript e não use outras ferramentas como `esbuild` ou `swc`. São ótimas alternativas, mas não suportam alguns dos recursos do TypeScript (ex. [a API Transformers](https://levelup.gitconnected.com/writing-typescript-custom-ast-transformer-part-1-7585d6916819)).
- O arquivo `tsconfig.json` existente deve gerenciar todas as configurações. 
- Se o código for executado em desenvolvimento, ele também deverá ser executado em produção. Ou seja, não use duas ferramentas de desenvolvimento e produção completamente diferente e depois ensine as pessoas a ajustar seu código.
- Adicione suporte leve para copiar arquivos estáticos para a pasta de compilação final. Normalmente, esses serão os modelos do Edge.
- Certifique-se de que o `REPL` também possa executar o código TypeScript como um cidadão de primeira classe. Todas as abordagens acima, exceto `ts-node`, não podem compilar e avaliar o código TypeScript diretamente.

## Compilador de desenvolvimento na memóra
Semelhante ao `ts-node`, criamos o módulo [@adonisjs/require-ts](https://github.com/adonisjs/require-ts). Ele usa a API do compilador TypeScript, o que significa que todos os recursos do TypeScript funcionam, e seu arquivo `tsconfig.json` é a única fonte de verdade.

No entanto, `@adonisjs/require-ts` é ligeiramente diferente do `ts-node` das seguintes maneiras.

- Não realizamos nenhuma verificação de tipo durante o desenvolvimento e esperamos que você conte com seu editor de código para o mesmo.
- Armazenamos a [saída compilada](https://github.com/adonisjs/require-ts/blob/develop/src/Compiler/index.ts#L185-L223) em uma pasta de cache. Portanto, da próxima vez que seu servidor reiniciar, não recompilamos os arquivos inalterados. Isso melhora a velocidade drasticamente.
- Os arquivos em cache precisam ser excluídos em algum momento. O módulo `@adonisjs/require-ts` expõe os [métodos auxiliares](https://github.com/adonisjs/require-ts/blob/develop/index.ts#L43-L57) que o observador de arquivos AdonisJS usa para limpar o cache do arquivo alterado recentemente.
- A limpeza do cache é essencial apenas para reivindicar o espaço em disco. Não afeta o comportamento do programa.

TOda vez que você executa `node ace serve --watch`, iniciamos o servidor HTTP junto com o compilador na memória e observamos o 
sistemas de arquivos quanto a alterações nos arquivos.

## Builds de produção independentes
Você cria seu código para produção executando o comando `node ace build --production`. Ele executa as seguintes operações.

- Limpa o diretório de compilação existente (se houver)
- Cria seus assets do front-end usando o Webpack Encore (somente se estiver instalado).
- Usa a API do compilador TypeScript para compilar o código TypeScript para JavaScript e gravá-lo dentro da pasta de compilação. Desta vez, realizamos a verificação de tipo e relatamos os erros do TypeScript.
- Copia todos os arquivos estáticos para a pasta de compilação. Os arquivos estáticos são registrados dentro do arquivo `.adonisrc.json` no array `metaFiles`.
- Copia o `package.json` e o `package-lock.json/yarn.lock` para a pasta `build`.
- Gera o arquivo `ace-manifest.json`. Ele contém um índice de todos os comandos que seu projeto está usando.
- Isso é tudo.

### Por que chamamos isso de build autônoma?
Depois de executar o comando `build`, a pasta de saída tem tudo o que você precisa para implementar seu aplicativo em produção.

Você pode copiar a pasta `build` sem seu código-fonte do TypeScript e seu aplicativo funcionará bem.

Criando uma pasta `build/` autônoma ajuda a reduzir o tamanho do código que você implementa em seu servidor de produção. Isso geralmente é útil quando você empacota seu aplicativo como uma imagem do Docker. No entanto, não há necessidade de ter a fonte e a saída de compilação em sua imagem do Docker e mantê-la leve.

### Pontos para levar em conta
- Após a compilação, a pasta de saída se torna a raiz do seu aplicativo JavaScript.
- Você deve sempre dá `cd` para a pasta `build/` e então rodar seu app.
```
cd buid
node server.js
```
- Você deve instalar dependências somente de produção dentro da pasta `build/`.
```
cd build
npm ci --production
```
- Não copiamos o arquivo `.env` para a pasta de saída. Como as variáveis de ambiente não são transferíveis, você deve definir as variáveis de ambiente para produção separadamente.

<footer align="center">
  <a href="./04-Environment_variables.md">Voltar para Environment Variables ------------------------</a>
  <a href="./06-Deployment.md">Ir para Deployment</a>
</footer>