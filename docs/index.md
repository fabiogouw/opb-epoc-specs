# Open Banking Brasil - Encaminhamento de Proposta de Crédito

Esta é uma proposta de especificação do produto a ser entregue na fase 3C do Open Banking, que trata de propostas de crédito que utilizam os dados compartilhados dos clientes para a geração de propostas mais assertivas.

## Premissas

## Fluxo

![Fluxo de negócio envolvendo o cliente, o correspondente digital, a IF receptora e a IF transmissora](./imgs/fluxo.png)

## Máquina de Estados do Pedido

![Máquina de estados do pedido de proposta de crédito](./imgs/maquina-de-estados-pedido-proposta.svg)

## APIs Envolvidas

### API no Correspondente Digital

Dado que o Correspondente Digital não faz parte do ecossistema do Open Banking, nenhuma API deve ser exposta por este participante. Há questões de segurança, que são necessárias implementar, que inviabilizam a sua manutenção pelo Correspondente. Além disso, qualquer API no Correspondente ficaria de fora de testes funcionais (Motor de Conformidade).

### API na Instituição Financeira Receptora

#### Encaminhamento de Pedido de Proposta

Request: POST /credit-proposals

```json
{
    "bankCorrespondentId": "00000000000000",  // identificação do correspondente digital
    "proposalRequestId":"00000000000000-6be83e91-4e54-4b10-9704-e781bca001c1",    // identificação da requisição da proposta de crédito, gerado unicamente pelo correspondente
    "providers": [
        "0cf6dd54-0a00-4ee2-8af3-71fc9ed1da85", // campo AuthorisationServerId que identifica a marca no diretório
        "53da2398-c211-457c-bf65-d1b233f89f61",
    ],  // identificação das IFs transmissoras (que poderão dar o consentimento / compartilhamento)
    "businessEntity": {
        "document": {
            "identitication": "11111111111",
            "rel": "CPF"
        }
    },  // identificação do cliente
    "permissionsRequired": [
        "RESOURCES_READ",
        "CUSTOMERS_PERSONAL_IDENTIFICATIONS_READ",
        "ACCOUNTS_READ",
        "ACCOUNTS_BALANCE_READ",
    ],
    "detail": {
        "productType": "EMPRESTIMOS",
        "productSubType": "CREDITO_PESSOAL_SEM_CONSIGNACAO", // modalidade do 3040
        "amountRequired": 5000, // montange desejado
        "instalmentMaxValue": 200,  // valor máximo de parcela
        "interestMaxValue": 10,  // taxa máxima
        "period": 2000, // prazo
        "duration": 1000, // duração da operação
        "gracePeriod": 100  //prazo de carência
    }
}
```

Response (201 Created)

```json
{
    "data": {
        "creditProposalId": "a88631a9-ab68-4fce-af0a-87107d4b64d4"  // identificação da requisição da proposta de crédito
    },
    "links": {
        "self": "https://api.banco.com.br/open-banking/api/v1/credit-proposals/a88631a9-ab68-4fce-af0a-87107d4b64d4"
    },
    "meta": {
        "requestTime": "2021-11-30T09:41:23"
    }
}
```

#### Consulta do Pedido de Proposta

Request: GET /credit-proposals/{id}

```json
Identificação do correspondente digital
Identificação da requisição da proposta de crédito
Identificação das IFs transmissoras (que darão o consentimento)
Status da proposta (em análise, sem interesse, disponível)
```

Response

```json
```

#### Consulta de Proposta Gerada

Request GET /credit-proposals/{id}/proposals

```json
Identificação do correspondente digital
Identificação da requisição da proposta de crédito
Identificação do encaminhamento de proposta de crédito
Caracteristica da proposta disponibilizada
    - modalidade da operação
    - tipo da operação
    - montante total a ser pago
    - valor da parcela
    - prazo da operação (qtd de parcelas)
    - taxa mensal efetiva de juros
    - periodicidade
    - tarifas
    - seguros
    - IOF
    - encargo por atraso
    - CET
    - validade da proposta
    - vencto da primeira parcela e da última
    - valor fixo ou variável
    - índice da correção da parcela
    - uso de dados compartilhados (sim ou não)
    - condições comerciais
    - moeda
```

Response

```json
```

#### Atualização de Uso da Proposta

Request PATCH /credit-proposals/{id}/proposals/{id}

```json
Identificação do correspondente digital
Identificação da requisição da proposta de crédito
Identificação do cliente
Status do aceite (aceito ou rejeitado) + data e hora
```

Response

```json
```

### API na Instituição Financeira Transmissora

#### Pedido de Consentimento em Lote