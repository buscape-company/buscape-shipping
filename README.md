Buscapé Marketplace

# API Cotação - V1

Definição da API de cotação de frete a ser implementada pelos vendedores do Buscapé.

### 1 - Definição

* A cotação é solicitada pelo Buscapé através de um endpoint provido pelo vendedor.
* A API deve ser desenvolvida em REST, seguindo as especificações da tecnologia.
* A autenticação utilizará o método de token.

### 2 - Autenticação

O token de autorização será vinculado com o perfil do vendedor no buscapé.

Junto de todas as chamadas via api, será incluso no header o parâmetro _**Authorization**_ junto com o token vinculado ao seller.

Exemplo: 

```text
POST /quote/seller/112211
Host: {host_do_vendedor}
Content-Type: application/json; charset=utf-8
Authorization: Token 3dbcf7bccc4c4aaaab45a9a05675e9eb
Cache-Control: no-cache
```

### 3 - Consultas de cotações

A consulta de cotação é feita a partir do envio de uma lista de itens contendo o SKU do item do seller e o cep de destino.

##### - HTTP Request Method: POST

##### - HTTP Request URL Pattern: _Definido pelo vendedor_

Obs. Caso a url seja de um marketplace, defina o vendedor na url do mesmo para que esteja disponível via GET do HTTP.
Ex: /quote/seller/112211 ou  /quote/?seller=112211


##### - Json Body Request: 

```json
{ 
   "zipcode":"00000000", 
   "items":[ 
      { 
         "sku":"SKU_PARCEIRO_1", 
         "quantity":2 
      }, 
      { 
         "sku":"SKU_PARCEIRO_2", 
         "quantity":1 
      } 
   ] 
} 
```

| Campo | Formato | Descrição |
|-------|---------|-----------|
| zipcode | string | Cep de destino |
| items | array[object] | Lista de items para cotação |
| items[].sku | string | Sku do item do vendedor |
| items[].quantity | int | Quantidade a ser enviada do item atual|

##### - Json Body Response:  

Na resposta esperamos um array de cotações para os items enviados. 

```json
{   
   "id":123456,
   "shipping":[   
      {   
         "items":[   
            {   
               "sku":"SKU_PARCEIRO_1",
               "quantity":2
            },
            {   
               "sku":"SKU_PARCEIRO_2",
               "quantity":1
            }
         ],
         "quotes":[   
            {   
               "price":13.00,
               "isScheduledDelivery":false,
               "estimatedShippingDate":"2017-05-18",
               "deliveryTime":{   
                  "estimate":3,
                  "shipping":2,
                  "handling":1
               },
               "method":{   
                  "id":1111,
                  "name":"Sedex"
               }
            },
            {   
               "price":15.00,
               "isScheduledDelivery":false,
               "estimatedShippingDate":"2017-05-16",
               "deliveryTime":{   
                  "estimate":1,
                  "shipping":1,
                  "handling":0
               },
               "method":{   
                  "id":1221,
                  "name":"Sedex 10"
               }
            },
            {   
               "price":20.00,
               "isScheduledDelivery":true,
               "scheduledDeliveries":[   
                  {   
                     "date":"2017-05-16",
                     "shift":[   
                        "morning",
                        "afternoon"
                     ]
                  },
                  {   
                     "date":"2017-05-17",
                     "shift":[   
                        "morning"
                     ]
                  },
                  {   
                     "date":"2017-05-18",
                     "shift":[   
                        "night"
                     ]
                  }
               ],
               "method":{   
                  "id":1331,
                  "name":"Entrega Agendada"
               }
            }
         ]
      }
   ]
}
```

| Campo | Formato | Obrigatório | Descrição |
|-------|---------|-------------|-----------|
| id | int | sim | Identificador da cotação no vendedor |
| shipping | array[object] | sim | Lista de cotações |
| shipping[].items | array[object] | sim | Lista de itens da entrega |
| shipping[].items[].sku | string | sim | Sku contemplado na entrega |
| shipping[].items[].quantity | int | sim | Quantidade a ser enviada do item atual |
| shipping[].quotes | array[object] | sim | Lista de cotações|
| shipping[].quotes[].price | float | sim | Valor da cotação |
| shipping[].quotes[].isScheduledDelivery | bool | sim | Se a entrega é agendada ou normal |
| shipping[].quotes[].estimatedShippingDate | date | sim/não | Obrigatório sempre que não entrega agendada |
| shipping[].quotes[].deliveryTime | object | sim/não | Estimativa de entrega para o item atual (Obrigatório se isScheduledDelivery for falso) |
| shipping[].quotes[].deliveryTime.estimate | int | sim/não | Estimativa de tempo de entrega em dias*  (Obrigatório se isScheduledDelivery for falso) |
| shipping[].quotes[].deliveryTime.shipping | int | não | Estimativa do tempo de transporte em dias* |
| shipping[].quotes[].deliveryTime.handling | int | não | Estimativa do tempo de empacotamento em dias* |
| shipping[].quotes[].method | object | sim | Informações do método de envio |
| shipping[].quotes[].method.id | int | sim | Identificador do método no vendedor. (deve ser um id único, para que o vendedor saiba qual método e cotação escolhidos a partir desse id) |
| shipping[].quotes[].method.name | string | sim | Nome do método |
| shipping[].quotes[].scheduledDeliveries | array[object] | sim/não | Datas disponíveis para agendamento de entrega. (Obrigatório se isScheduledDelivery for verdadeiro) |
| shipping[].quotes[].scheduledDeliveries[].date | string | sim | Data disponível para entrega no formato YYYY-MM-DD |
| shipping[].quotes[].scheduledDeliveries[].shift | array[string] | não | Turnos disponíveis para essa data de entrega. Valores disponíveis: "morning", "afternoon", "night" |

\* Dias de entrega devem ser definidos em dias úteis

##### - HTTP Status esperado:

| Código | Descrição |
|--------|-----------|
| 200    | Sucesso |
| 400    | Erro na validação dos parametros enviados |
| 404    | Iten(s) não encontrado(s) |



# API Rastreio - V1 

### 1 - Definição

API para receber os status de envio do pedido.

Sempre que houver uma mudança de estado nesse processo (shipping),deve ser enviada uma atualização para o Buscapé.

### 2 - Autenticação

Segue a mesma estrutura de autenticação da API de cotação, com o vendedor enviando o token de autenticação para o Buscapé.
Utilizaremos o mesmo token já vinculado ao vendedor para fazer essa autenticação.

Exemplo:

```text
POST /api/tracking
Host: {host_do_buscape}
Content-Type: application/json; charset=utf-8
Authorization: Token 3dbcf7bccc4c4aaaab45a9a05675e9eb
Cache-Control: no-cache
```

### 3 Envio do rastreio

Estados processados por esta API:

| Ordem | Estado | Código |
|:-----:|--------|--------|
| 1 | Faturado | invoiced |
| 2 | Na transportadora | in_hosting | 
| 3 | Em rota de entrega | in_route |
| 4 | Devolvido | reversal |
| 5 | Retentativa | retrying |
| 6 | Completo | complete |

1. **invoiced**: Antes de enviar o tracking, é obrigatório enviar a nota fiscal. Ex:
    ```json
    { 
       "orderId":"123", 
       "status":"invoiced", 
       "invoice":[{ 
          "items":[   
           {   
               "sku":"SKU_PARCEIRO_1",
               "quantity":2
           }],
          "number":"b413bfe83d704c76bd4f81f99abf30c9", 
          "value":250.90, 
          "url":"https://buscape.slack.com/messages/D6X2MSFPE/", 
          "issuanceDate":"2017-09-09", 
          "key":"fe83d704c76bd4f" 
       },
       { 
           "items":[   
           {   
               "sku":"SKU_PARCEIRO_2",
               "quantity":1
           }],
          "number":"5f04b770edda4da6ad0f4723e186dd91", 
          "value":100.90, 
          "url":"https://buscape.slack.com/messages/GFGCDFXFD/", 
          "issuanceDate":"2017-09-09", 
          "key":"bb83456x46xbd4f" 
       }]
    } 
    ```
    
2. **in_hosting**: Com o pedido na transportadora, atualiza o estado junto ao código de rastreio. Ex:
    ```json
    { 
       "orderId":"123", 
       "status":"in_hosting", 
       "items":[   
           {   
               "sku":"SKU_PARCEIRO_1",
               "quantity":2
           }],
       "url":"http://www.correios.com.br/rastreio/BR625252S",
       "tracking":"BR625252S"
    } 
    ```
    
3. **in_route**, **reversal** e **retrying**: São estados de atualização das informações para o cliente, para que o mesmo possa saber como está o processo de entrega do seu pedido. Ex:
    ```json
    { 
       "orderId":"123", 
       "status":"in_route"
    } 
    ```
    
4. **complete**: Quando todo o pedido tiver sido entrege, deve-se enviar o estado de completo para que possamos finalizá-lo.
    ```json
    { 
       "orderId":"123", 
       "status":"complete"
    } 
    ```

##### - HTTP Request Method: POST

##### - HTTP Request URL Pattern: _HOST/api/tracking_

##### - Json Body Request:

```json
{ 
   "orderId":"123", 
   "status":"invoiced", 
   "invoice":[{ 
        "items":[{   
            "sku":"SKU_PARCEIRO_1",
            "quantity":2
        }],
       "number":"b413bfe83d704c76bd4f81f99abf30c9", 
       "value":250.90, 
       "url":"https://buscape.slack.com/messages/D6X2MSFPE/", 
       "issuanceDate":"2017-09-09", 
       "key":"fe83d704c76bd4f" 
    },
    { 
        "items":[{   
            "sku":"SKU_PARCEIRO_2",
            "quantity":1
        }],
       "number":"5f04b770edda4da6ad0f4723e186dd91", 
       "value":100.90, 
       "url":"https://buscape.slack.com/messages/GFGCDFXFD/", 
       "issuanceDate":"2017-09-09", 
       "key":"bb83456x46xbd4f" 
    }]
} 
```

| Campo | Formato | Obrigatório | Descrição |
|-------|---------|-------------|-----------|
| orderId | string | sim | Identificador do pedido no Buscapé |
| status | string | sim | Estado de controle do pedido* |
| invoice | object | sim/não | Dados da nota fiscal (Obrigatório caso o campo **status** for **invoiced**) |
| invoice.items | array[object] | sim | Lista de itens da fatura |
| invoice.items[].sku | string | sim | Sku contemplado na fatura |
| invoice.items[].quantity | int | sim | Quantidade do item faturado |
| invoice.number | string | sim | Número da nota fiscal |
| invoice.value | float | sim | Valor total da nota fiscal |
| invoice.url | string | não | Url da nota fiscal |
| invoice.issuanceDate | string | não | Data de emissão da nota fical. Formato: YYYY-MM-DD |
| invoice.key | string | não | Chave de acesso da nota fiscal |
| url | string | não | URL de rastreio caso o campo  **status** for **in_hosting** |
| tracking | string | não | Código de rastreio caso o campo  **status** for **in_hosting** |

\* Vide tabela de estados acima

##### - Json Body Response:

```json
{
  "success": true
}
```

| Campo | Formato | Descrição |
|-------|---------| ----------|
| success | bool |  |
| message | string | Mensagem de resposta (caso houver) |

##### - HTTP Status:

| Código | Descrição |
|--------|-----------|
| 200    | Sucesso |
| 400    | Erro na validação dos parametros enviados |
| 404    | Pedido não encontrado |
