# Colmena: Teste prático para a vaga de desenvolvedor Javascript Fullstack

> ![image](https://user-images.githubusercontent.com/17680680/178080912-16d54714-2c3c-4d89-b11a-8a16d3b4af54.png)
>
> A Colmena é uma caixa de ferramentas digitais para produções da mídia local e comunitária. É uma solução 100% gratuita, segura e de código aberto para ambientes móveis e desktop. https://blog.colmena.media/

## Instruções
Você deverá criar um `fork` deste projeto, e desenvolver em cima do seu fork. Use o *README* principal do seu repositório para nos contar como foi resolver seu teste e as decisões tomadas, e principalmente as instruções de como rodar seu projeto, afinal a primeira pessoa que irá rodar seu projeto será um programador de nossa equipe.

Lembre-se que este é um teste técnico e que não existe apenas uma resposta correta. Indicaremos, ao longo deste documento, algumas ferramentas e tecnologias que estão mais alinhadas com nossa realidade no time, porém não seremos tão rígidos em suas escolhas técnicas, mas claro, como premissa da vaga, considere frameworks ou bibliotecas em Javascript para chegar na solução final. Estipularemos um prazo máximo de entrega de **4 dias**.

## O desafio
Você irá construir um painel administrativo que, basicamente, buscará informações de algumas API's distintas para listar o conteúdo na tela, além de outras manipulações que serão explicadas com mais detalhes ao longo deste documento. É uma aplicação simples, e poderemos dividir o esforço em 3 etapas:

 1. Fluxo de autenticação: Formulário simples para entradas de dados (usuario e senha) relacionados a autenticação.
 2. Listagem de conversas: Após a autenticação, esta será a primeira página que o sistema deverá exibir. Será uma listagem simples das conversas retornadas da API.
 3. Listagem de publicações: Esta página será acessada através de um menu de navegação, após a autenticação do usuário. Será uma listagem de publicações retornadas da API.
 
 Vamos explicar com mais detalhes cada uma dessas três etapas citadas. O importante agora é entender um pouco melhor sobre as duas APIs que deverão ser utilizadas no teste. A primeira seria a API do [Nextcloud](https://nextcloud.com/). Ele é um software para armazenamento, sincronização e compartilhamento de arquivos, e como já o utilizamos em nosso projeto principal, seria interessante considerá-lo neste teste. A segunda API seria um headless CMS chamado [Strapi](https://strapi.io/). Ele é um Framework de Gerenciamento de Conteúdo (CMF), que nos oferece facilidades no desenvolvimento de um CMS, e no caso se aplicaria muito bem a este contexto de publicações para usuário externos, sendo também um framework importante no nosso cenário real do projeto. 
 
 ### 1. Fluxo de autenticação
 Esperamos uma ideia de interface simples para a autenticação. O CSS desta tela e do resto da aplicação não será um item muito importante para avaliação, por isso, não se preocupe em fazer nada muito rebuscado de layout para as telas. 
 Para autenticar, usaremos um _endpoint_ específico da API do Nextcloud para isso. Exemplo abaixo:
 
 ```javascript
const axios = require('axios');

const config = {
  method: 'get',
  url: 'https://claudio.colmena.network/ocs/v2.php/cloud/user?format=json',
  headers: { 
    'OCS-APIRequest': 'true', 
    'Authorization': 'Basic Y29saWVyYTpjb22tZW5hMTIa'
  }
};

axios(config)
.then(function (response) {
  console.log(JSON.stringify(response.data));
})
.catch(function (error) {
  console.log(error);
});
 ```
 
 Este _endpoint_ pode retornar dois códigos de status: 200 em caso de sucesso e 401 em caso de falha.
 
<details><summary>Retorno para o código 200:</summary>

<p>

```json
   {
    "ocs": {
        "meta": {
            "status": "ok",
            "statuscode": 200,
            "message": "OK"
        },
        "data": {
            "storageLocation": "/var/www/html/data/colmena",
            "id": "colmena",
            "lastLogin": 1657449922000,
            "backend": "Database",
            "subadmin": [],
            "quota": {
                "free": 58333175808,
                "used": 57,
                "total": 58333175865,
                "relative": 0,
                "quota": -3
            },
            "avatarScope": "v2-federated",
            "email": "colmena@colmena.media",
            "emailScope": "v2-federated",
            "displaynameScope": "v2-federated",
            "phone": "",
            "phoneScope": "v2-local",
            "address": "",
            "addressScope": "v2-local",
            "website": "",
            "websiteScope": "v2-local",
            "twitter": "",
            "twitterScope": "v2-local",
            "groups": [
                "Radio_Colmena_EN"
            ],
            "language": "pt_BR",
            "locale": "",
            "backendCapabilities": {
                "setDisplayName": true,
                "setPassword": true
            },
            "display-name": "Javascript Tester"
        }
    }
}
```

</p>

</details>

<details><summary>Retorno para o código 401:</summary>

<p>

```json
  {
    "ocs": {
        "meta": {
            "status": "failure",
            "statuscode": 997,
            "message": "Current user is not logged in"
        },
        "data": []
    }
}
```

</p>

</details>

Podemos perceber que o _endpoint_ 200 não retorna _token_ de autenticação. O ciclo de vida da sessão do usuário deverá ser implementada no framework Javascript escolhido. Já utilizamos por aqui o Next.js, que resolve isso pra gente. Porém, fique à vontade para utilizar outro de sua escolha. **O usuário e senha para a autenticação serão enviados para seu e-mail**.  

### 2. Listagem de conversas
O Nextcloud prover um serviço de chat, através de uma API chamada [Talk](https://nextcloud-talk.readthedocs.io/en/latest/). E basicamente o que vamos fazer nesta tela será uma listagem dessas conversas. 

Exemplo da requisição:

```javascript
const axios = require('axios');

const config = {
  method: 'get',
  url: 'https://claudio.colmena.network/ocs/v2.php/apps/spreed/api/v1/room?format=json',
  headers: { 
    'OCS-APIRequest': 'true', 
    'Authorization': 'Basic Y29sbWV0YTpjb3xtZW5hMT2z'
  }
};

axios(config)
.then(function (response) {
  console.log(JSON.stringify(response.data));
})
.catch(function (error) {
  console.log(error);
});

```

Para entender o retorno deste _enpoint_, favor consultar a documentação da [API de Talk](https://github.com/nextcloud/spreed/blob/master/docs/conversation.md) do Nextcloud.

Na interface, esperamos ver as seguintes informações:
 - Nome da conversa (displayName)
 - Quantidade de mensagens não lidas (unreadMessages)
 - Se está marcada como favorito (isFavorite)
 - Última mensagem enviada (lastMessage.message)
 
Será uma listagem simples, sem a necessidade de componentes de busca e nem paginação. 
 
### 3. Listagem de publicações
Esta listagem utilizará o Strapi como fonte de consulta. Este framework é feito em Nodejs e sua instalação é bastante simples, como podemos ver em seu [Github oficial](https://github.com/strapi/strapi). A ideia é que você possa subir uma instância do Strapi local para fazer o desenvolvimento, não tendo a necessidade de enviá-lo como artefato da entrega. Nós já teremos uma instância do Strapi por aqui, e no caso só faríamos o redirecionamento para o nosso quando formos testar seu código.

Sobre a estrutura do Strapi, teremos uma _Collection_ chamada Publication, com os seguintes campos:
 - uuid: ID único para uma publicação
 - title: Título 
 - description: Descrição
 - author: Autor
 - status: Status da publicação (booleano)
 - creation_date: Data de criação
 
Os registros para a _Collection_ _Publication_ poderão ser inseridos via interface administrativa do Strapi. O que esperamos na interface desta página é uma listagem destas publicações, com a opção de alterar o status de cada registro. Este status representaria a visibilidade ou não de uma publicação para o usuário final.

Abaixo um exemplo das interfaces de criação da _Collection_ no Strapi, com os tipos de cada coluna:

![tela_1_publicacao](https://user-images.githubusercontent.com/17680680/178144470-dec9aaea-3b21-41ab-8609-30ad05ffaad6.jpg)

![tela_2_publicacao](https://user-images.githubusercontent.com/17680680/178144477-43d6fd10-3c4e-40e7-8a3d-56cc0862d051.jpg)
 
## O que esperamos do seu teste

- Os requisitos mínimos implementados: Autenticação, tela da listagem de conversas e a tela de listagem de publicações.
- Ver na solução a utilização de um framework Javascript. Utilize o framework da melhor forma possível (metodologia/estrutura).
- Utilização de um framework CSS no desenvolvimento. Utilizamos o [Material-UI](https://mui.com/), porém fique a vontade para utilizar outro se preferir.
- Um HTML escrito da maneira mais semântica possível (HTML5)
- Não tem necessidade de aplicar design responsivo neste projeto
- Boas práticas de desenvolvimento (Clean Code)

## O que nós ficaríamos felizes de ver em seu teste
- Testes unitários
- Protótipagem das telas em alguma ferramenta de criação de interface (Ex: Figma)

## O que nos impressionaria
- Aplicar o GraphQl na comunicação com o Strapi
- Documentar a arquitetura da solução através de diagramas, considerando as aplicações envolvidas na solução.

## O que nós não gostaríamos
- Descobrir que não foi você quem fez seu teste
- Ver commits grandes, sem muita explicação nas mensagens em seu repositório
- Alguma solução sem utilizar o Javascript como base

## O que avaliaremos de seu teste
- Alcance dos objetivos propostos
- As instruções de como rodar o projeto
- Organização, semântica, estrutura, legibilidade, manutenibilidade do seu código
- Componentização e extensibilidade dos componentes Javascript
