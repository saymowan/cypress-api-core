## 🚀 Cypress 4 test automation api 🚀

![Cypress 4 test automation api](https://github.com/saymowan/cypress-api-core/workflows/API%20Rest%20tests/badge.svg)
[![Code Quality](https://www.code-inspector.com/project/20271/score/svg)](https://frontend.code-inspector.com/project/20271/dashboard)
[![Badge ServeRest](https://img.shields.io/badge/API-ServeRest-green)](https://github.com/PauloGoncalvesBH/ServeRest/)
[![Cypress.io](https://img.shields.io/badge/tested%20with-Cypress-04C38E.svg)](https://www.cypress.io/)

Projeto para estudo e definição de uma arquitetura base para testes automatizados de API Rest com [Cypress](https://www.cypress.io/).

### ✊ Uso deste material
-----------------------
- Seja maneiro (a), faça referência ao utilizar esta arquitetura/repositório ✌️;


### ✨ Instalação e uso da arquitetura
-----------------------
- Instale o [Node.js](https://nodejs.org/en/download/);
- Baixe este repositório ou faça um git clone;
- Abra o diretório do projeto e execute o comando:
    - `npm install`
- Para abrir a interface de execução do Cypress, execute no diretório do projeto:
    - `npx cypress open`
- Próximo passo é configurar/instalar a API ServeRest na sua máquina;


### ✨  Instalação API ServeRest
-----------------------
- A nossa API alvo deste projeto é a ServeRest localmente, para utiliza-la execute a aplicação [via npm](https://www.npmjs.com/package/serverest) ou [via Docker](https://hub.docker.com/r/paulogoncalvesbh/serverest/). 
- Para mais detalhes visite o [repositório oficial do ServeRest](https://github.com/ServeRest/ServeRest).


### ⚙️ Arquitetura do projeto
-----------------------

```
cypress4testautomationapi/
  ├─  cypress/
  │        │
  │        ├── fixtures/
  │        │   ├── *.json
  │        │   ├── *.csv       
  │        │   └── *.png
  │        │
  │        ├── integration/
  │        │   ├── <categoria>/
  │        │   │   └── <categoria>Tests.spec.js
  │        │   └── <categoria2>/
  │        │       └── <categoria2>Tests.spec.js
  │        │
  │        ├── plugins/
  │        │   └── index.js
  │        │
  │        ├── reports/
  │        │   └── mocha/
  │        │         └── mochafiles (*.json, *html)
  │        │
  │        ├── support/
  │        │   ├── databaseCommands.js
  │        │   ├── apiGeneralCommands.js
  │        │   ├── api<Categoria>Commands.js
  │        │   ├── api<Categoria2>Commands.js
  │        │   └── index.js
  │        │  
  │        └── videos/
  │ 
  ├── environmentsConfig/
  ├── node_modules/
  ├── cypress.json
  ├── package-lock.json
  ├── package.json
  └── README.md
```

---------------------------------------
## 🔍 Camadas da arquitetura

 - **fixtures:** arquivos para massa de dados estática para os testes (csv, png, xlsx, txt);
 - **integration:** arquivos de testes separados em categorias/módulos da API para facilitar a organização. Extensão *.spec.js;
 - **plugins:** plugins que são utilizados na solução ficam dentro do arquivo "plugins/index.js";
 -  **reports:** diretório com o relatório de execução dos testes usando Mocha Awesome;
 - **support:** camada com comandos Cypress customizados e sobrescritas globais:
    - Mapeamento das requisições (headers, requestservice, parametros [body, path, queryString]) para reuso em diferentes testes.
    - Arquivo para comandos de select/insert em banco de dados.
    - Arquivo index.js responsável de receber as importações dos comandos Cypress;
 - **videos:** geração opcional de videos das execução dos testes;
 - **environmentsConfig:** diretório com os arquivos de configuração por ambiente;
 - **node_modules:** arquivos ou diretórios que podem ser carregados pelo Node.js;
 - **cypress.json:** arquivo de configuração do Cypress;
 - **package-lock.json:** gerado automaticamente com as instalações e atualizações de pacotes;


### 💡 Features
-----------------------
<details><summary><i>Requests como commands</i></summary>
Cada endpoint é mapeado com a sua estrutura (headers, parâmetros, método, endpoint, cookies) no Cypress commands para focarmos em reuso. Os arquivos de mapeamento de requisições podem ser feitos por módulo/categoria.
Exemplo:

![Exemplo requisição](https://i.imgur.com/ctY5Zkv.png)

No exemplo vemos o mapeamento do endpoint Produtos para ser usado por todos os testes de API que desejam utiliza-lo.
Para criar um teste com esta requisição basta utilizar o command referente e passar o(s) parametro(s):

```js
    it('Produtos - Buscar Produto Inexistente', ()=>{
        cy.getProdutos('nome=9dj9128dh12h89')
            .then(response =>{
            expect(response.status).to.equal(200)
            expect(response.body.quantidade).to.equal(0)
        })
    })
```

</details>

<details><summary><i>Testes isolados e e2e</i></summary>
Testes de requisição de maneira isolada para validar parâmetros válidos, inválidos, status code estão presentes nesta arquitetura:

```js
    it('Produtos - Excluir Produto Inexistente',()=>{

        cy.deleteProdutos("xxx", true)
            .then(response =>{
                expect(response.status).to.equal(200)
                expect(response.body.message).to.eq("Nenhum registro excluído")
            })            
    })
```

Testes de múltiplas requisições (e2e) podem ser feitos com esta arquitetura, veja exemplo de um teste para Deletar um Produto (produto é criado durante o teste):

```js
it('Produtos - Excluir Produto Existente',()=>{

    const produto ={
        nome: faker.random.uuid(),
        preco: faker.random.number(),
        descricao: "Mouse bom",
        quantidade: "5"
        }

    cy.postProdutos(produto)
        .then(response =>{
        expect(response.status).to.equal(201)
        expect(response.body.message).to.equal("Cadastro realizado com sucesso")
        let _id = response.body._id

            cy.deleteProdutos(_id, true)
                .then(respDelete =>{
                    expect(respDelete.status).to.equal(200)
                    expect(respDelete.body.message).to.eq("Registro excluído com sucesso")
                })   

                cy.getProdutos('_id='+_id)
                .then(respGet =>{
                    expect(respGet.status).to.equal(200)
                    expect(respGet.body.quantidade).to.equal(0)
                })              
            })
    })
```
</details>

<details><summary><i>Testes de exceção de status code (4xx e 5xx)</i></summary>

Para testes de exceção de status code (client side [4xx] or server side [5xx]) precisamos incluir um parâmetro [failOnStatusCode](https://docs.cypress.io/api/commands/request.html#Arguments) na requisição com valor false.

Vide exemplo de mapeamento de requisição:

```js
Cypress.Commands.add('deleteProdutos', (productId, failStatusCode) =>{
    cy.api({
        method: 'DELETE',
        url: '/produtos/'+productId,
        headers: {  Authorization : localStorage.getItem('token') },
        failOnStatusCode: failStatusCode
    })
})
```

Vide exemplo de teste "forçando" um erro para validar o statuscode e response body:

```js
    it('Produtos - Excluir Produto token expirado',()=>{
        localStorage.setItem('token', "token erradinho")

        cy.deleteProdutos("xxx", false)
            .then(response =>{
                expect(response.status).to.equal(401)
                expect(response.body.message).to.eq("Token de acesso ausente, inválido, expirado ou usuário do token não existe mais")
            })            
    })
```

</details>

<details><summary><i>Mock de dados</i></summary>

[Biblioteca Faker](https://github.com/marak/Faker.js/) para mock de dados. 
Vide [exemplos de dados](https://github.com/marak/Faker.js/#api-methods) que podem ser mascarados.

</details>

<details><summary><i>Data Driven Testing</i></summary>

A arte de reaproveitar o mesmo teste com o mesmo fluxo e asserção variando somente a massa de teste proveniente de dados estáticos ou arquivos (*.csv, *.json, *.xlsx), chamamos de Data Driven Testing ([leia mais sobre](https://medium.com/@saymowan/data-driven-testing-ddt-e-o-reaproveitamento-dos-testes-automatizados-8c8d67cc211c)), na arquitetura temos o uso de um arquivo json (JArray) para a massa de testes:

```json
[
    {
        "nome": "Mouse Gamer Adamantiun Shinigami Usb",
        "preco": 98,
        "descricao": "Mouses para Jogos",
        "quantidade": 12
    },
    {
        "nome": "Monitor Gamer AOC Agon 32'' Curvo 165Hz",
        "preco": 269,
        "descricao": "Monitores Gamer",
        "quantidade": 45
    },
    {
        "nome": "Kit 3 Roteadores Gigabit Wifi TP-Link Rede Mesh AC1200",
        "preco": 189,
        "descricao": "Dispositivos de Conexão em Rede",
        "quantidade": 78
    }
]
```

O mesmo teste é criado N vezes através do arquivo json:

```js
const produtos = require('../../fixtures/Produtos/produtosList.json')
const faker = require('faker')

  //JArray (produtoList.json) com cada objeto a ser cadastrado
  produtos.forEach(produto => {
  it('Produtos - Cadastrar Produto DDT',()=>{

      let expectedStatusCode = 201;
      let expectedSuccessMessage = "Cadastro realizado com sucesso";

      const produtoTestData ={
          "nome": produto.nome + "-" + faker.random.number(),
          "preco": produto.preco,
          "descricao": produto.descricao,
          "quantidade": produto.quantidade
        }

      cy.postProdutos(produtoTestData)
          .then(response =>{
          expect(response.status).to.equal(expectedStatusCode)
          expect(response.body.message).to.equal(expectedSuccessMessage)            
      })
  })
})
```


</details>

<details><summary><i>Mocha report customizado</i></summary>

Em desenvolvimento

</details>

<details><summary><i>Chai: asserção status code e response body</i></summary>

Podemos validar de diversas formas os dados retornados no response (body, cookies, headers, status code), vide exemplos que podem ser aplicados: [clique aqui](https://docs.cypress.io/guides/references/assertions.html#BDD-Assertions).

Exemplo de assertiva de status code e parâmetro "message" do response body com o método "to.equal":

```js
    expect(response.status).to.equal(expectedStatusCode)
    expect(response.body.message).to.equal(expectedSuccessMessage)            
```

</details>


<details><summary><i>Orquestração de métodos</i></summary>

A organização dos métodos que devem ser executados antes ou depois dos testes ou bateria pode ser feito através de méetodos nativos do Cypress, [clique aqui para detalhes](https://docs.cypress.io/guides/core-concepts/writing-and-organizing-tests.html#Hooks).

Um exemplo comum para testes de API é a geração de token de acesso a cada teste, veja exemplo abaixo do método que é executado antes de cada teste para garantir o acesso dos recursos com o token correto:

```js
    beforeEach(() => {
        cy.generateTokenAsAdmin()
    })
```

Neste caso, o Token é gerado como admin e usamos a Request as Command (../support/apiGeneralCommands.js), veja a requisição mapeada e já enviando o token para o storage para ser usado por todos os testes no header:

```js
Cypress.Commands.add('generateTokenAsAdmin', () =>{
    cy.api({
        method: 'POST',
        url: '/login',
        body: {
            "email": "fulano@qa.com",
            "password": "teste"
          }
    })
    .then(response =>{
        expect(response.status).to.eql(200)
        localStorage.setItem('token', response.body.authorization)
        expect(localStorage.getItem('token')).not.null
        cy.log(localStorage.getItem('token'))
    })      
})
```

</details>


<details><summary><i>Arquivo de configuração</i></summary>

Recurso nativo do Cypress através do arquivo cypress.json. [Vide documentação oficial](https://docs.cypress.io/guides/references/configuration.html#Options).
</details>

<details><summary><i>Variáveis globais por ambiente</i></summary>

Para modificar suas variáveis globais por ambiente temos uma pasta criada "cypress/environmentsConfig" com dois possíveis ambientes "Prod" e "Qa" representados pelos arquivos exampleProd.json e exampleQa.json respectivamente.

Foi feita a inclusão de um plugin (/cypress/plugins/index.js) através do método "getConfigurationByFile()" onde podemos alterar o ambiente ao executar pela linha de comando incluindo qual ambiente se deseja:

```
npx cypress run --env configfile=exampleProd
```

</details>


<details><summary><i>Geração e uso de token</i></summary>

Vide feature "Orquestração de métodos" para entender como o Token é gerado/orquestrado. Para o uso basta incluir o header na Request as Command e incluir o item do localStorage "token":

```js
Cypress.Commands.add('postProdutos', bodyJson =>{
    cy.api({
        method: 'POST',
        url: '/produtos',
        body: bodyJson,
        headers: {  Authorization : localStorage.getItem('token') }})
})
```

</details>

<details><summary><i>Parametros via Json, QueryString e Path</i></summary>

### Path

Exemplo de uso de parâmetro Path com a requisição Delete Produtos:

![Delete Produtos ServeRest](https://i.imgur.com/yQVpCwt.png)

Ao mapear a requisição (Request as Command) incluímos o parâmetro junto ao request service (parâmetro url):

```js
Cypress.Commands.add('deleteProdutos', (productId, failStatusCode) =>{
    cy.api({
        method: 'DELETE',
        url: '/produtos/'+productId,
        headers: {  Authorization : localStorage.getItem('token') },
        failOnStatusCode: failStatusCode
    })
})

```

### QueryString

Exemplo de uso de parâmetro QueryString com a requisição Get Produtos:

![Get Produtos ServeRest](https://i.imgur.com/0x8pzuC.png)

Ao mapear a requisição (Request as Command) incluímos o parâmetro junto ao request service (parâmetro url) devendo ser informado quais parâmetros concatenados na camada de testes (integration):

```js
Cypress.Commands.add('getProdutos', queryString =>{
    cy.api({
        method: 'GET',
        url: '/produtos?'+ queryString})
})
```

Este recurso ainda está em pesquisa para ser otimizado.

### Json

Exemplo de uso de parâmetro Json com a requisição Post Produtos:

![Post Produtos ServeRest](https://i.imgur.com/jNS8H3t.png)

Neste caso temos um json de envio, com os seguintes parâmetros:

```json
{
  "nome": "nome",
  "preco": "1",
  "descricao": "descricao",
  "quantidade": "1"
}
```

Ao mapear a requisição (Request as Command) incluímos o parâmetro "body" com nossa estrutura de json "jsonBody". Nossos dados virão da camada de testes (integration):

```js
Cypress.Commands.add('postProdutos', jsonBody =>{
    cy.api({
        method: 'POST',
        url: '/produtos',
        body: jsonBody,
        headers: {  Authorization : localStorage.getItem('token') }}) // header de autenticação
})
```

Camada de teste com o envio dos dados no teste, video a constante "produto" com os dados mockados:

```js
    it('Produtos - Cadastrar Produto',()=>{

        const produto ={
            "nome": faker.random.uuid(),
            "preco": faker.random.number(),
            "descricao": "Mouse bom",
            "quantidade": "5"
          }

        cy.postProdutos(produto)
            .then(response =>{
            expect(response.status).to.equal(201)
            expect(response.body.message).to.equal("Cadastro realizado com sucesso")            
        })
    })
})
```

</details>

<details><summary><i>Pipeline de teste via Github Actions</i></summary>

Pipeline feito com Github Actions executado em máquina Linux com os processos:

- Instancia da aplicação ServeRest local via Docker
- Execução de todos os testes - Task nativa Cypress 

Flows executados [disponível aqui](https://github.com/saymowan/cypress-api-core/actions);

Arquivo yml [disponível aqui](https://github.com/saymowan/cypress-api-core/tree/master/.github/workflows);

</details>


### 🆕 Novas features 
---------------------------
Para novas features [crie uma issue](https://github.com/saymowan/cypress-api-core/issues/new) ou verifique o [board do projeto](https://github.com/saymowan/cypress-api-core/projects/1).


### 🌟 Contribuições
--------------------------
Para novas contribuições, faça um fork do projeto, realize as alterações e submeta um Pull Request ou [crie uma issue](https://github.com/saymowan/cypress-api-core/issues/new) para ser avaliada;
