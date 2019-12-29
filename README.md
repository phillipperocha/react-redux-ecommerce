# Arquitetura Flux um Ecommerce

## Vamos construir um ecommerce para aprender o Flux com a implementação do Redux, usando Redux Saga para fazer o side effects das funcionalidades assíncronas

Vamos construir um ecommerce do zero bem bonito com Create React App utilizando o Redux, veja as imagens no final do post.

### Utilizaremos o Redux para explicar o Flux.

## Aula 01 - Conceitos de Redux

- Biblioteca que implementa Arquitetura Flux;
  - Ela pode ser utilizada com qualquer framework javascript (Angulas, Vue, React), ou até mesmo com JS puro
  - Redux implementa o Flux, [Flux](https://facebook.github.io/flux/) é uma arquitetura, um conceito que facilita a comunicação entre os elementos em tela.
- Controle de estados globais da aplicação, um estado global é quando não tem um dono específico, ele é pode ser usado em qualquer componente ou estrutura de código para renderizar uma tela;
- Quando utilizar o Redux?
  - Meu estado tem mais de um “dono”? Quando o estado é exibindo em outros lugares, não apenas em um componente específico.
  - Meu estado é manipulado por mais componentes? Se sim, podemos utilizar o Redux.
  - As ações do usuário causam efeitos colaterais nos dados? Exemplo: adicionar um produto no carrinho, deveria disparar uma mensagem para o usuário ou mudar um contador em outro componente da tela.

Redux é utilizado para controlar estados globais, um bom exemplo para utilizar Redux: Carrinho de compras, dados do usuário logado com as permissões, player de música, etc;

Quando o estado não ter um dono específico ou tiver que ser exibido em outros lugares, então o Redux é uma boa solução, geralmente para médio a grande projetos onde os estados se espalham na aplicação.

## Arquitetura Flux

Sempre que queremos acessar ou atualizar um estado nós disparamos uma Ação (Action)

Em um e-commerce, se o usuário clica em catálogo de produtos, um action é disparada (a action é disparada não apenas quando o usuário faz uma ação, pode ser o próprio sistema disparando uma ação, por exemplo quando o componente carrega no `componentDidMount` do React, então podemos disparar um `action` para fazer algo)

A Action tem uma estrutura:

```json
{
  type: ADD_TO_CART,
  product: { ... } 
}
```

Que é um tipo e o payload, o valor que ela carrega ou retorna.

Então no nosso caso, estamos enviando uma ação de Adicionar ao carrinho, um produto que o usuário gostou e escolheu.

Essa ação vai para o `Redux Store`, que contém os `reducers` (termo que designa a separação de estados no Redux Store) no redux podemos ter vários estados, e cada reducers separa o estado por funcionalidade, podemos ter um Store com vários reducers. Podemos ter um Cart Reducer, User Reducer, onde Cart armazena os valores de compra e User armazena os valores do usuário logado, etc. No exemplo acima, o Cart Reducer recebe a action ADD*TO*CART, ele faz uma mutação no estado, alterando o valor, incluindo no Carrinho um novo produto.

E agora o componente Carrinho no header da aplicação escuta a alteração, e ele mesmo pode disparar uma action para atualizar o valor de quantidade inserida no carrinho:

```json
{
  type: UPDATE_QUANTITY,
  product: { ... },
  quantity: 5
}
```

Um reducer recebe essa action e faz a alteração no estado novamente e esse valor é replicado para quem ouvir esse estado.

## Princípios

- Toda action deve possuir um "type", informando o tipo da ação e deve ser uma string única.
- O estado do Redux é o único ponto de verdade, se o estado do carrinho estiver no Redux todo os dados referente ao carrinho tem que estar no Redux, não pode ficar no componente e no Redux ao mesmo tempo, isso ocorrer gera inconsistência e fica ruim para manter.
- Não podemos mudar o estado do Redux sem um action; O estado é imutável, apenas o Reducer pode alterar de acordo com as chamadas da Action.
- As actions e reducers são funções puras, ou seja, não lidam com side-effects assíncrons, isso quer dizer que, elas não vão acessar banco de dados, chamar api, etc. Isso ajuda muito com testes unitários, pois uma função pura, sempre que receber o mesmo parâmetro sempre vai devolver o mesmo resultado. Para lidar com side-effects assíncronos usamos Redux Saga que veremos mais pra frente.
- Qualquer lógica síncrona para regras de negócio deve ficar no reducer e nunca na action, a action não altera dados, ela só carrega os valores, o reducer faz a mutação do estado
- Nem toda aplicação precisa Redux, inicie sem ele e sinta a necessidade depois, é uma das forms mais fáceis de entender Redux, pq colocar Redux de cara, vc vai escrever mais código e não vai entender porque está sendo útil, só é legal colocar de cara se você souber que o projeto vai crescer, se os requisitos estiverem bem claros;

## Exemplo do Carrinho

Cart Reducer, sempre vai iniciar com o estado vazio, um `[ ]` vazio por exemplo.

O Cart Reducer sempre vai ouvir as actions que a aplicação disparar, se ele ouvir o tipo da action, ele faz o que precisa fazer, então o Cart Reducer ouve a action do Type: ADD*TO*CART:

```json
{
  type: "ADD_TO_CART",
  product: {
    id: 1,
    title: "Novo produto",
    price: 129.9
  }
}
```

E Agora o Cart Reducer colocar o produto no estado:

```json
[
  {
    id: 1,
    title: "Novo produto",
    price: 129.9,
    amount: 1,
    priceFormatted: "R$129,90"
  }
]
```

O reducer é capaz de adicionar campos mesmo sem a action tem enviado os campos, veja o `amount` e `priceFormatted`. O reducer sabe que vou precisar do preço formatado então já criou isso pra gente e como é o primeiro produto que coloco no carrinho, logo a quantidade é 1. E essa quantidade vai ser usado em algum lugar, então temos esses dados a nossa disposição.

E outro momento o usuário pode escolher o mesmo produto de ID 1, e então apenas atualizamos o valor de quantidade.

Chamando outra action do type: UPDATE_AMOUNT, com produto de ID 1 e com quantidade 5.

```json
{
    type: “UPDATE_AMOUNT”,
    product: 1,
    amount: 5,
}
```

O Cart Reducer ouve essa actions e manipula o estado da maneira necessidade:

```json
[{
    id: 1,
    title: "Novo produto",
    price: 129.9,
    amount: 5,
    priceFormatted: "R$129,90"
}]
```

Pronto, agora atualizou apenas o valor.

Redux é fácil, basta entender o conceito de Actions, Store e Reducers, e principalmente quando utilizar o Redux na aplicação.

## Aula 02 - Estrutura do Projeto

Vamos criar uma loja virtual de calçados (Rocketshoes) para aprender a implementação do Redux.

Utilizaremos o Create React App para criar o frontend da aplicação em React:

```shell
npx create-react-app rocketshoes
```

Executei no terminal:

```shell
yarn & yarn start
```

Pronto, tudo rodando!

## Aula 03 - Configurando Rotas

Vamos criar a configuração de navegação do projeto.

Não importei o `BrowserRouter` de dentro do `routes.js` pois iremos criar um componente Header que também precisará de ter acesso aos dados da rota.

Então os componentes que precisarão de Rotas vão ficar no App.js

## Aula 04 - Estilos Globais

- Instalamos a lib styled-components;
- Criamos o arquivo globals.js com os estilos globais;
- Importamos o estilo para App.js

## Aula 05 - Criando o Header

- Crio o componente de estilização do header
- Crio o componente Header que tem uma logo e link para a home e o carrinho e o link para o carrinho
- Coloco o Header no topo do App.js

## Aula 06 - Estilização da Home

- Criamos a estilização no arquivo styled.js e aplicamos no index.js da Home
- Destaque para a lib [polished](https://polished.js.org/) que instalamos, com `yarn add polished`. Agora podemos manipular as cores com essa lib.

https://www.youtube.com/watch?v=CbFMwHvHK1Y

## Aula 07 - Estilização do Carrinho

- Criamos a estilização no arquivo styled.js e aplicamos no index.js do Cart
- Utilizamos novamente a lib do polished para gerenciar a cor do hover do botão finalizar pedido.

## Aula 08 - Configurando a API

Configurar a API para consumir produtos.

Utilizamos a API [json-server](https://github.com/typicode/json-server) para criar uma API Fake, que simula o que aconteceria em uma API Real.

Só criar um json com os dados fake e rodar o json-server que vai simular a API, inclusive dá para configurar tempo de resposta e outras coisas mais.

Para instalar de forma global na máquina:

```text
yarn add json-server -D
```

Depois criar um arquivo `server.json` na raiz do projeto e colocar os dados contendo o estoque (stock) e os produtos (products), inclusive fazendo os relacionamentos com id.

Teremos uma rota stock que retorna o estoque dos produtos, e a rota products que retorna os produtos.

Vamos configurar o axios para consumir essa api.

```shell
yarn add axios
```

Veja o arquivo `api.js` no código fonte.

E para rodar a api fake, basta rodar:

```shell
yarn json-server server.json -p 3333 -w
```

E podemos também ter um script no package.json:

```json
...
"server":  "json-server server.json -p 3333 -w"
...
```

E rodar `yarn server` para executar a api.

- json-server é o nome da lib
- server.json é o nome do arquivo que tem a api fake e está na raiz do projeto, por isso não passo o caminho, apenas o nome do arquivo. Está no mesmo nível do package.json
- -p 3333 é a porta que defini para rodar essa api
- -w é para ficar `watching` cada alteração que eu fizer na api, se eu mudar alguma coisa lá dentro não preciso rodar o comando novamente.

Pronto agora para acessar, só chamar a rota que deseja:

```text
http://localhost:3333/stock
```

e

```text
http://localhost:3333/products
```

Ambas as rotas vão mostrar um array com seus respectivos objetos.

Legal, que se eu chamar o produto e passando o id, ele me traz apenas o produto com seu respectivo id informado:

```
[http://localhost:3333/products/4](http://localhost:3333/products/4)
```

Se eu passar um ID q não exisite ele retorna um objeto vazio.

Dá até para deletar e atualizar valores dentro dessa api, é muito legal!

Excelente para usar em modo de desenvolvimento no frontend quando um backend não foi feito ainda com a API.

## Aula 09 - Buscando produtos da API

Nessa aula vamos buscar os dados da API e renderizar na tela do usuário, seguindo algumas boas práticas de código.

- Transformar o componente stateless Home em uma classe statefull para poder armazenar o estado dos products que vieram da API.
- Criei um arquivo `format.js` dentro de `src/util` que exporta uma função que formata valores em moeda brasileira, utilizando a instância de `Intl.NumberFormat` do Javascript.
- Buscamos os dados da API, armazenamos no estado products, e populamos a lista de produtos na tela
- Assim que buscamos os dados da API fizemos um map para devolver um array de produtos com o preço formatado, ao invés de usar a função `formatPrice()` de dentro do render na variável price do product. Não é recomendado colocar funções que manipulam variáveis de dentro do render, pois a cada alteração no estado, essa função é chamada também, fazendo com que perca performance, a ideia é sempre entregar para o render os valores prontos para serem renderizados e qualquer formatação deve ser feita antes.