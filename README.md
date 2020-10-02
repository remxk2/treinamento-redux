### Para os interessados, eu gravei uma call na qual tive com o pessoal explicando sobre quase todos os pontos abordado aqui, [link](https://web.microsoftstream.com/video/a5667ea9-d641-40d1-8b0d-e4c624491728).

# Primeiro uma pergunta simples, o que é Redux?

[Redux](https://redux.js.org/) nada mais é que uma ferramenta para controle de estado da aplicação para o _react/reactive-native_. 

# 1. O que é estado?
Controle de estado é como os dados devem ficar guardados na aplicação, todos os dados que estão dentro da `store` que é como chamamos o local estado ficam guardados. 
Por sua natureza estão os dados na `store` estão disponiveis para toda a aplicação trazendo o beneficio imediato  na qual o componente X pode se comunicar com o Y mesmo que ambos não tenha nenhuma relação entre si.

### Os dois tipos de estados.

No redux nós temos dois tipos de estados:
- **Temporario**: este é o comum, os dados permanacem na `store` independente da navegação do usuario, ou seja, todos os dados vão persistir durante aquela sessão e serão perdidos apenas quando o usuario fechar o app (não minimizar) ou fechar a pagina no caso web.
- **Persistentes**: com a ajuda da lib [react-persist](https://github.com/rt2zz/redux-persist) nós podemos criar uma `store` persistente na aplicação fazendo com que os dados sejam mantidos na  mesmo se o usuario fechar o app/navegador. 
No caso web o desenvolvedor tem a opção de utilizar qualquer sistema de permancencia de dados com o plugin, de cookies até database do navegador, normalmente se é usado localStorage para web e AsyncStorage para o mobile

### Qual estado devo usar?

O estado **persistente** só deve ser usado caso realmente haja a necessidade  de permanecer os dados mesmo que o usuario feche o navegador. O caso mais comum para persistir é armazenar o token de autenticação do usuario ou outras informações como historicos de busca, caso contrario a **temporaria** é sempre a preferivel

# 2. Você não vai falar sobre o que são os *Sagas*?

### Beleza, vamos lá, o que são *Sagas* ?

O [redux-saga](https://redux-saga.js.org/) é um middleware de monitoramento do redux, ele não tem conexão com os *reducers* (na qual vamos falar mais frente) e seu funcionamento é independente do mesmo.

### Pra que serve?

Os Sagas foram desenvolvidos para facilitar os efeitos colaterais de chamadas asyncronas tornando-as mais faceis de trabalhar, mais eficiente para executar e melhor para lidar com falhas.

A ideia geral é que o saga é rodado em uma thread separada da sua aplicação cujo unico proposito é lidar com os efeitos dessa chamada asyncrona o que significa que a thread pode ser iniciada, pausada e até cancelada pela propria aplicação usando o redux.

Sagas tem acesso a toda a store da aplicação.


# 3. Duck Pattern

Pra quem não quer ler, nosso colega Diego da *rocketseat* fez um [excelente video](
https://www.youtube.com/watch?v=q-If9n-tUyA) bem resumido sobre duck patterns (e até outros pontos abordados nesse post) e até um [blog post](https://blog.rocketseat.com.br/estrutura-redux-escalavel-com-ducks/).

De forma resumida, Duck patterns é uma alternativa para a arquitetura do redux, normalmente o redux usa uma arquitetura dividida por modules que suas actions, reducers e types.

```// store/auth/actions.js
// store/auth/reducers.js
// store/auth/types.js

// store/friends/actions.js
// store/friends/reducers.js
// store/friends/types.js

// store/posts/actions.js
// store/posts/reducers.js
// store/posts/types.js
```

Esta arquitura tem um lado ruim de ficar muito separado, e sempre que voce for criar um modulo novo vai precisar criar os arquivos separados e suas devidas importações.

**Duck patterns** veio para solucionar este problema, ele junta os arquivos em 1 só, chamado *duck*

```
// store/ducks/auth.js
// store/ducks/friends.js
// store/ducks/posts.js
```


*Não irei entrar em detalhes como funciona ou como configurar um duck por enquanto pois iremos fazer isso na sessão do redux-sauce*

# 4. Reduxsauce, agora a magica acontece.

[Este carinha aqui](https://github.com/jkeam/reduxsauce), é onde toda a magica vai acontecer e como que ele torna o duck pattern muito mais atraente e resolve a coisa mais feia do redux **switch case** 😨 .

Basicamente o **reduxsauce** vai juntar e se encarregar de criar todos os creators, types, reducers, e ainda suporta integração com typescript mesmo que nao completamente.

![](upload://8imZv4c9HCCkfZVOcG5hZdnf847.png)
**Reduxsauce a melhor sauce.**

*Iremos entender como criar e configurar mais a frente*

# 5. Reactotron, mais coisa ainda??

[Reactotron](https://github.com/infinitered/reactotron) é um aplicativo de terceiro que consegue monitorar todas as chamadas de api, actions, reducers e sagas, dando para o desenvolvedor uma interface muito boa a intituiva na qual podemos observar todo o comportamento do redux e chamadas exteriores da aplicação.

![1_qVD2bVCUvl-FF2OGeh6KaA1|600x500](upload://sQz5dvkZarHByI8BB30PkRzxtWn.png) 

# 6. Agora é a hora, hora de juntar tudo. Como configurar? 🤔
*Pensando em como explicar tudo isso.*
![](upload://yiOeuzdwXspKZMU87Q81IhASjjK.gif)

Não vou entrar em muitos detalhes como sobre como configurar mas sim irei explicar as configurações existentes.

## 6.1 - Baby steps, primeira coisa, configurar a store inicial.

Aqui esta nossa config que pode ser localizada no caminho `src\store\index.ts` do nosso APP
```typescript

// Configuração do objeto inicial na qual o plugin react-persist pede.
const persistConfig = {
  whitelist: ['persist', 'auth'], // Nomes dos ducks (nome que foi registrado), nos quais os dados serão persistentes
  key: 'root',
  storage: AsyncStorage, // Tipo de storage
};

// Verificação de env, caso for env de produção não precisamos usar o reactotron e suas depedencias
if (process.env.NODE_ENV === 'development') {
  // eslint-disable-next-line global-require
  require('@/config/reactotron');
}

type RootState = ReturnType<typeof reducers>;

// Passamos a config do persist que criamos e os reducers registrados (iremos ver mais a frente)
const persistedReducer = persistReducer(persistConfig, reducers);

// Aqui iremos adicionar os middlewares, no nosso caso Sagas e Reactotron, novamente fazendo a checagem de env.
const middlewares = [];

const sagaMonitor =
  process.env.NODE_ENV === 'development' && typeof console.tron
    ? console.tron.createSagaMonitor!()
    : {};

const sagaMiddleware = createSagaMiddleware({ sagaMonitor });

middlewares.push(sagaMiddleware);

// Aqui a configuração que junta os middlewares
const composer =
  process.env.NODE_ENV === 'development' && typeof console.tron
    ? compose(
        composeWithDevTools(
          applyMiddleware(...middlewares),
          console.tron.createEnhancer!(),
        ),
      )
    : compose(applyMiddleware(...middlewares));

// Finalmente criamos a store, aqui fomos obrigado a tipar ela pois nos estamos criando ela atraves do
// persistor, e o persistor tende a dar conflito de tipagems então um rapido fix é assim
export const store = createStore<RootState & PersistPartial, any, any, any>(
  persistedReducer,
  composer,
);
export const persistor = persistStore(store);

sagaMiddleware.run(sagas);
```

`composeWithDevTools` vem de uma extensão chamada [Redux DevTools](https://github.com/zalmoxisus/redux-devtools-extension) ela nos permite debugar de forma similar ao *Reactotron*

## Linkando os componentes com a store

Agora nós precisamos englobar nossa aplicação inteira com o `Provider` do redux e o `PersistGate` do *react-persist*, usando o persit e o store que criamos nas config, o modo mais facil de fazer isso é colocar ele envolta do AppRoot, no nosso caso se chama `Navigate`

`src\index.tsx`
```typescript
import { store, persistor } from './store';

const App: React.FC = () => (
  <Provider store={store}>
    <PersistGate loading={null} persistor={persistor}>
      <StatusBar
        translucent={true}
        backgroundColor="rgba(255, 255, 255, 0)"
        barStyle="light-content"
      />
      <Navigator />
    </PersistGate>
  </Provider>
);

export default App;
```

Pronto agora todos seus componentes do App tem acesso a sua store, simples não?

## Tava simples, agora vem o monstro BIR----

Aqui irei fazer passo a passo a criação de um *duck*, pois não tem como explicar ele sem ir passo a passo.

Exemplo a seguir iremos criar o *duck* de produtos.

### Primeira coisa, tipagem... achou que ia escapar né?

Primeira coisa, criar os tipos, não do typescript, nas do redux. O redux escuta as *actions* atras dos types definidos, isso que iremos criar agora.

#### Types

`src\interfaces\product.ts`
```typescript
export enum ProductEnum {
  REQUEST_PRODUCT_DETAILS = 'REQUEST_PRODUCT_DETAILS',
  REQUEST_PRODUCT_DETAILS_SUCCESS = 'REQUEST_PRODUCT_DETAILS_SUCCESS',
  REQUEST_PRODUCT_DETAILS_FAILURE = 'REQUEST_PRODUCT_DETAILS_FAILURE',
  REQUEST_PRODUCT_INFO = 'REQUEST_PRODUCT_INFO',
  REQUEST_PRODUCT_INFO_SUCCESS = 'REQUEST_PRODUCT_INFO_SUCCESS',
  REQUEST_PRODUCT_INFO_FAILURE = 'REQUEST_PRODUCT_INFO_FAILURE',
  SET_ALL_PRODUCT_FILTERS = 'SET_ALL_PRODUCT_FILTERS',
  SET_CURRENT_FILTER = 'SET_CURRENT_FILTER',
  SET_PRODUCT_FILTER = 'SET_PRODUCT_FILTER',
  REMOVE_PRODUCT_FILTER = 'REMOVE_PRODUCT_FILTER',
  RESET_FILTERS = 'RESET_FILTERS',
}
```
Aqui fazemos uso do `enum` do typescript pois é mais trabalhar com ele do que objetos.
Voce pode notar que existe um certo padrão de 3 ali e voce está correto, este padrão de `REQUEST` `REQUEST_SUCCESS` e `REQUEST_FAILED` é mais usado quando estamos criando ações que irão usar o **saga**, pois uma requsição só pode ter 2 resultados, sucesso ou falha.

Os outros como não estão usando *sagas* não há necessidade para criamos o padrão de 3.

Mas vale notar que existem casos que a regra pode ser seguida mas de maneira diferente pois usando a logica, quando voce insere um objeto na store voce tambem vai querer remover não é?
Por isso temos ali `SET_PRODUCT_FILTER ` para adicionar e `REMOVE_PRODUCT_FILTER` para remover.

Pronto, criamos nossso tipos e automaticamente criamos a interface dele como vou te mostrar logo abaixo.

#### Actions Types
Agora aqui é preciso **prestar muita atenção** em um pequeno detalhe, os types do redux seguem por pardão de regra a nomenclatura igual escrevemos ali `TUDO_MAIUSCULO_COM_UNDERLINE_EM_VEZ_DE_ESPAÇO` e é imperativo seguir esta regra a risca pois é assim que o **reduxsauce** vai esperar la na frente quando formos declarar as actions e não os tipos.

`src\interfaces\product.ts`
```typescript
export interface IProductActionTypes extends Action<ProductEnum> {
  requestProductDetails: (id: number) => Action;
  requestProductDetailsSuccess: (
    data: IProductRequestDetailsSuccessResponse,
  ) => Action;
  requestProductDetailsFailure: () => Action;
  requestProductInfo: (id: number) => Action;
  requestProductInfoSuccess: (data: IProductRowInfo[]) => Action;
  requestProductInfoFailure: () => Action;
  setAllProductFilters: (filters: IFilter2[]) => Action;
  setProductFilter: (filter: SelectedFilter) => Action;
  removeProductFilter: (filter: IFilter2) => Action;
  setCurrentFilter: (filter: IFilter2) => Action;
  resetFilters: () => Action;
}
```
Reparou uma coisa? Como os nomes das Actions seguem o padrão dos types porem em **camelcase**?

**Exemplo:** `REQUEST_PRODUCT_DETAILS ` virou `requestProductDetails`

Isto não é atoa, pois quando o **reduxsauce** receber os types, eles serão convertidos para **camelcase** então um errinho de texto ali pode te dar uma dor de cabeça imensa.

Então com o nome da Action explicada, o parametro dela é uma função, cujo parametro é o que esta Action irá receber quando for chamada.

**Exemplo:** `requestProductDetails` recebe `(id: number) => Action` por padrão todos eles devem retornar `Action` mas o payload dela é o desenvolvedor quem decide, nesta ação estamos enviando apenas o `id` do produto nele.

Nos outros nos estamos enviando `Filter2` que é uma interface e assim por diante.

#### Criando os Types do enum

`src\interfaces\product.ts`
```ts
export type ProductTypes = typeof ProductEnum;
```

Simples!

#### Interfaces de payload
Aqui devemos criar a interface do payload na qual o *reducer*  ira receber quando a action for disparada, isso é um pequeno ponto negativo do **reduxsauce** não consegue receber apenas o ID, ele recebe um payload, por isso precisamos declarar. 
Obs: Vai ficar mais claro quando criamos os reducers

Então vamos criar a interface do `requestProductDetails`

`src\interfaces\product.ts`
```ts
export interface IProductRequestDetails {
  id: number;
}
```
Bem simples né? Apenas uma interface que tem como unico parametro o atributo que foi passado pela action.

#### Initial State
Por padrão nossa store deve tambem ser tipada, todos os itens dentro dela, qual os valores ela suporta, quais os tipos do mesmo, quais valores são iniciais, pois sim, devemos ter valores iniciais como regra para evitar problemas de `map of undefined` e similares.

`src\interfaces\product.ts`

```ts
export interface IProductStateTypeInitialData extends LoadingState {
  product: IProductDetails | null;
  productInfo: IProductRowInfo[] | null;
  filters: IFilter2[];
  selectedFilters: SelectedFilter[];
  currentFilter: IFilter2 | null;
}
```

Este LoadingState é uma interface generica global que que obriga o estado inicial ter os loadings, pois lembra que nos sagas nos controlamos chamadas asyncronas? Então com os loadings nos temos controle de... bem... dos loadings... Vai fazer mais sentido nos reducers, #confia.

`src\interfaces\app.ts`
```ts
export interface LoadingState {
  loadings: {
    get: boolean;
    post: boolean;
    put: boolean;
    delete: boolean;
  };
}
```

Pronto, tipagem concluida, nem doeu né?

### Finalmente, para criação dos ducks 👉
Agora iremos fazer uso das tipagems criadas anteriormente e criaremos nossas actions e reducers,


#### Actions e Types
Agora vem um dos beneficios do **reduxsauce**, ele vai criar sozinho o tipo e actions

`src\store\ducks\products.ts`
```ts
/* ------------- Types and Actions ------------- */

export const {
  Types: ProductCreatorTypes,
  Creators: ProductCreators,
} = createActions<ProductTypes, IProductActionTypes>({
  requestProductDetails: ['id'],
  requestProductDetailsSuccess: { product: '' },
  requestProductDetailsFailure: [],
  requestProductInfo: ['id'],
  requestProductInfoSuccess: ['data'],
  requestProductInfoFailure: [],
  setAllProductFilters: ['filters'],
  setProductFilter: ['filter'],
  removeProductFilter: ['filter'],
  setCurrentFilter: ['filter'],
  resetFilters: [],
});
```

Esta reparando os nomes? Eles precisam seguir as regras dos Types declarado no nosso `enum`, por isso era de extrema importancia nossa tipagem das Actions seguir o mesmo padrão, pois pode notar que a função `createActions` recebe 2 genericos, os `ProductTypes` e as `IProductActionTypes` e é assim que conseguimos integrar com o typescript, agora nosso `Types` e `Creators` que são retornados da função  `createActions` estão devidamente tipados.

Voce pode notar que temos algo peculiar aqui nesta linha

```ts
export const {
  Types: ProductCreatorTypes,
  Creators: ProductCreators,
} 
```

Isto é apenas uma forma de renomer o nomes da variaveis, fazemos isso pois como teremos varios ducks, iria dar conflito de nomes caso exportassemos apenas como `Types` e `Creators`, por isso renomeados para `ProdutCreatorTypes` e `ProductCreators`

Mas agora voce se pergunta, "ok mas o que são aqueles arrays e objetos dentro da definição das actions? 

É aqui que fica bastante confuso.

No caso dos arrays, voce pode notar que temos ali `requestProductDetails: ['id']`, significa o que? Que o reduxsauce vai colocar o payload que esta vindo dentro de um **objeto** com a chave `id`, por isso criamos aquela interface que tinha como unico parametro `id: number`, lembra?
Então ao chamar a action `(id: number)`, ele ira colocar esse id dentro de um objeto, que fica assim
```ts
{ 
   id: 123
}
```

Já no caso dos objetos, vai ficar em **construção** pois é um pouco mais delicado de explicar e preciso dar uma investigada.

#### Initial Data

Agora iremos definir realmente a data inicial do nosso duck, nao apenas os tipos, basta criar um objeto cuja tipagem é a mesma do tipo criado la nas  interfaces, assim:

Lembre-se sempre de tipar o objeto para você não esquecer nada e ter ajuda do intellisense :smiley: 
```ts
export const INITIAL_STATE: IProductStateTypeInitialData = {
  product: null,
  productInfo: [],
  filters: [],
  selectedFilters: [],
  currentFilter: null,
  loadings: {
    delete: false,
    get: false,
    post: false,
    put: false,
  },
};
```

#### Agora para os reducers 🦇

Os reducers são os caras que irão cuidar da logica das actions, quando uma **action** for disparada, o reducer será chamado, e cabe ao reducer decidir o que fazer com o payload que da action. Eles são os responsaveis parar alterar a logica da store daquele **duck**

Então aqui iremos continuar usando nosso exemplo do `REQUEST_PRODUCT_DETAILS`

Nosso primeiro reducer que sera disparado quando ouvir a action com o type de cima:
```ts
export const ProductRequestDetails = (
  state = INITIAL_STATE,
): IProductStateTypeInitialData => ({
  ...state,
  loadings: { ...state.loadings, get: true },
});
````

Lembra acima que declaramos o `INITIAL_STATE`? Então, por padrão o primeiro argumento do reducer é o atual estado do seu duck, nos damos um valor default para que caso não estiver inicializado, ele ira usar o `INITIAL_STATE`, mas caso ja tenhamos modificado a store, ele ira enviar o objeto atualizado e vai ignorar o valor padrão.

Em segundo sera o payload da nossa action... mas pera, esse reducer não esta recebendo nosso payload com o `id` do produto, por que?

Bom, como nos estamos usando sagas para esta chamada, apenas o **saga** precisa saber o `id`, o reducer tem acesso mas ele nao precisa guardar esta informação e não se preocupe, no proximo reducer iremos pegar o payload.

Por fim o reducer devera retornar o `state` atualizado com os novos valores, ali no caso só definimos o `loading => get` como `true`, ai podemos monitorar quando nossa aplicação esta fazendo requests e assim dar feedback para o usuario, legal né?


# Continua...
