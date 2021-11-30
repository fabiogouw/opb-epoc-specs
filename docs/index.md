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

Request: POST /credit-proposals/v1/credit-proposals

Exemplo:
```json
{
    "bankCorrespondentId": "00000000000000",  // identificação do correspondente digital
    "proposalRequestId":"00000000000000-6be83e91-4e54-4b10-9704-e781bca001c1",    // identificação da requisição da proposta de crédito, gerado unicamente pelo correspondente
    "businessEntity": {
        "document": {
            "identitication": "11111111111",
            "rel": "CPF"
        }
    },  // identificação do cliente
    "dataSharing": {
        "providers": [
            "0cf6dd54-0a00-4ee2-8af3-71fc9ed1da85", // campo AuthorisationServerId que identifica a marca no diretório
            "53da2398-c211-457c-bf65-d1b233f89f61",
        ],  // identificação das IFs transmissoras (que poderão dar o consentimento / compartilhamento)
        "permissionsRequired": [
            "RESOURCES_READ",
            "CUSTOMERS_PERSONAL_IDENTIFICATIONS_READ",
            "ACCOUNTS_READ",
            "ACCOUNTS_BALANCE_READ",
        ],
    },
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
        "self": "https://api.banco.com.br/open-banking//credit-proposals/v1/credit-proposals/a88631a9-ab68-4fce-af0a-87107d4b64d4"
    },
    "meta": {
        "requestTime": "2021-11-30T09:41:23"
    }
}
```

#### Consulta do Pedido de Proposta

Esta API é usada pelos correspondentes digitais para obter os detalhes e o status da requisição que eles fizeram a IF receptora (a que vai gerar a proposta).

A proposta é utilizar polling periódico para que essa comunicação seja feita, mas podemos considerar também a alternativa de uso de webhooks, de forma que a IF receptora possa avisar o correspondente digital que há alguma mudança no status, de forma a otimizar a comunicação.

Request: GET /credit-proposals/v1/credit-proposals/{id}

Exemplo:
```
GET /credit-proposals/v1/credit-proposals/a88631a9-ab68-4fce-af0a-87107d4b64d4
```

Response (200 ok)

```json
{
    "data": {
        "creditProposalId": "a88631a9-ab68-4fce-af0a-87107d4b64d4",
        "bankCorrespondentId": "00000000000000",  // identificação do correspondente digital
        "proposalRequestId":"00000000000000-6be83e91-4e54-4b10-9704-e781bca001c1",    // identificação da requisição da proposta de crédito, gerado unicamente pelo correspondente
        "businessEntity": {
            "document": {
                "identitication": "11111111111",
                "rel": "CPF"
            }
        },  // identificação do cliente
        "dataSharing": {
            "providers": [
                "0cf6dd54-0a00-4ee2-8af3-71fc9ed1da85", // campo AuthorisationServerId que identifica a marca no diretório
                "53da2398-c211-457c-bf65-d1b233f89f61",
            ],  // identificação das IFs transmissoras (que poderão dar o consentimento / compartilhamento)
            "permissionsRequired": [
                "RESOURCES_READ",
                "CUSTOMERS_PERSONAL_IDENTIFICATIONS_READ",
                "ACCOUNTS_READ",
                "ACCOUNTS_BALANCE_READ",
            ],
        },
        "status": "PROPOSTA_GERADA"
    },
    "links": {
        "self": "https://api.banco.com.br/open-banking/credit-proposals/v1/credit-proposals/a88631a9-ab68-4fce-af0a-87107d4b64d4",
        "proposals": "https://api.banco.com.br/open-banking/credit-proposals/v1/credit-proposals/a88631a9-ab68-4fce-af0a-87107d4b64d4/proposals"
    },
    "meta": {
        "requestTime": "2021-11-30T10:35:10"
    }
}
```

#### Consulta de Proposta Gerada

Request GET /credit-proposals/v1/credit-proposals/{id}/proposals

Exemplo:
```
GET /credit-proposals/v1/credit-proposals/a88631a9-ab68-4fce-af0a-87107d4b64d4/proposals
```

Response (200 ok)

```json
{
    "data": [
        {
            "proposalId": "91240d49-0504-412c-b942-65b34bb9f207",
            "productType": "EMPRESTIMOS",
            "productSubType": "CREDITO_PESSOAL_SEM_CONSIGNACAO", // modalidade do 3040
            "dataShareUsed": true,
            "propostaValidity": "2021-12-01T10:35:10",  // validade da proposta
            "detail": {
                "contractAmount": 5000, // montante total a ser pago
                "instalmentValue": 100, //valor da parcela
                "instalmentNumber": 50, // prazo da operação (qtd de parcelas)
                "effectiveMonthlyInterestRate": 1.34, // taxa mensal efetiva de juros
                "CET": 0.29,    // CET
                "settlementDate": "2018-01-15", // vencto da primeira parcela
                "dueDate": "2028-01-15",    // vencto da última parcela
                "currency": "BRL",   // moeda
                "instalmentPeriodicity": "MENSAL",  // periodicidade
                "interestRates": [
                    {
                        "taxType": "EFETIVA",
                        "interestRateType": "SIMPLES",
                        "taxPeriodicity": "AA",
                        "calculation": "21/252",
                        "referentialRateIndexerType": "PRE_FIXADO", // índice da correção da parcela
                        "referentialRateIndexerSubType": "TJLP",
                        "referentialRateIndexerAdditionalInfo": "Informações adicionais",
                        "preFixedRate": 0.6,    // valor fixo ou variável
                        "postFixedRate": 0,
                        "additionalInfo": "Informações adicionais"
                    }
                ],
                "contractedFees": [
                    {
                        "feeName": "Renovação de cadastro",
                        "feeCode": "CADASTRO",
                        "feeChargeType": "UNICA",
                        "feeCharge": "MINIMO",
                        "feeAmount": 100000.04,
                        "feeRate": 0.25
                    },
                    {
                        "feeName": "IOF",
                        "feeCode": "IOF_CONTRATACAO",
                        "feeChargeType": "UNICA",
                        "feeCharge": "MINIMO",
                        "feeAmount": 10.04,
                        "feeRate": 0
                    }
                ],  // tarifas
                "contractedFinanceCharges": [
                    {
                        "chargeType": "JUROS_REMUNERATORIOS_POR_ATRASO",
                        "chargeAdditionalInfo": "Informações adicionais sobre encargos.",
                        "chargeRate": 0.07
                    }
                ],  // encargo por atraso
                "insurance": [

                ],  // seguros
                "additionalInfo": "Informações adicionais"  // condições comerciais
            },
            "status": "PENDING"
        }
    ],
    "links": {
        "self": "https://api.banco.com.br/open-banking/credit-proposals/v1/credit-proposals/a88631a9-ab68-4fce-af0a-87107d4b64d4/proposals",
        "credit-proposal-detail": "https://api.banco.com.br/open-banking/credit-proposals/v1/credit-proposals/a88631a9-ab68-4fce-af0a-87107d4b64d4"
    },
    "meta": {
        "requestTime": "2021-11-30T10:35:10"
    }
}
```

#### Atualização de Uso da Proposta

Request PATCH /credit-proposals/v1/credit-proposals/{id}/proposals/{id}

Exemplo 
```
PATCH /credit-proposals/v1/credit-proposals/a88631a9-ab68-4fce-af0a-87107d4b64d4/proposals/91240d49-0504-412c-b942-65b34bb9f207
```

```json
{
    "status": "ACCEPTED"    // Status do aceite (aceito ou rejeitado)
}
```

Response (200 ok)

```json
{
    "data": {
        "proposalId": "91240d49-0504-412c-b942-65b34bb9f207",
        "status": "ACCEPTED"
    },
    "links": {
        "self": "https://api.banco.com.br/open-banking/api/credit-proposals/v1/a88631a9-ab68-4fce-af0a-87107d4b64d4/proposals/91240d49-0504-412c-b942-65b34bb9f207"
    },
    "meta": {
        "requestTime": "2021-11-30T09:41:23"
    }
}
```

### API na Instituição Financeira Transmissora

#### Pedido de Consentimento em Lote

Criação de consentimento /consents/v2/consents

```json
{
  "data": {
    "loggedUser": {
      "document": {
        "identification": "22222222222222",
        "rel": "CNPJ"
      }
    },
    "businessEntity": {
      "document": {
        "identification": "11111111111111",
        "rel": "CNPJ"
      }
    },
    "permissions": [
      "ACCOUNTS_READ",
      "ACCOUNTS_OVERDRAFT_LIMITS_READ",
      "RESOURCES_READ"
    ],
    "objective": {
        "type": "PROPOSTA_DE_CREDITO",
        "detail": {
            "totalOfDataProviders": 2
        }
    },
    "expirationDateTime": "2021-05-21T08:30:00Z",
    "transactionFromDateTime": "2021-01-01T00:00:00Z",
    "transactionToDateTime": "2021-02-01T23:59:59Z"
  }
}
```