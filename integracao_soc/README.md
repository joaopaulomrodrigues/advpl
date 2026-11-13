# Integração Protheus x SOC (TLPP)

Integração de saída (Protheus -> SOC) via REST/JSON para inclusão e alteração de cadastros de RH/SST.
Namespace: `custom.integration.soc`. Documentação de código no padrão **ProtheusDOC** (`/*/{Protheus.doc}`).

## Arquitetura

| Fonte | Classe | Responsabilidade |
|--|--|--|
| soc_result.tlpp | SocResult | Fábrica do JsonObject de retorno padronizado |
| soc_logger.tlpp | SocLogger | Log (FwLogMsg/ConOut) com máscara de token |
| soc_mapper.tlpp | SocMapper | De/Para (tabela ZSC) e formatação de dados |
| soc_client.tlpp | SocClient | Camada HTTP (FWRest) e tratamento de status |
| soc_service_base.tlpp | SocServiceBase | Fluxo comum: posiciona, monta payload, valida, envia |
| soc_services.tlpp | SocFuncionario, SocCargo, SocTurno, SocSetor, SocFilial, SocDesligamento | Payload de cada entidade |
| soc_main.tlpp | SocTeste (User Function) | Exemplo/teste |

Fluxo: `Service.incluir/alterar` -> `seekRecord` -> `buildPayload` (usa SocMapper) -> validação de `required()` -> `SocClient:send` -> `SocResult`.

## Entidades

| Serviço | Tabela | Recurso | Chave esperada (cChave) |
|--|--|--|--|
| SocFuncionario | SRA | funcionario | RA_FILIAL+RA_MAT |
| SocCargo | SRJ | cargo | RJ_FILIAL+RJ_FUNCAO |
| SocTurno | SR6 | turno | R6_FILIAL+R6_TURNO |
| SocSetor | SI3 | setor | I3_FILIAL+I3_CUSTO |
| SocFilial | SM0 | unidade | M0_CODFIL |
| SocDesligamento | SRG/SRA | afastamento | RG_FILIAL+RG_MAT |

## Parâmetros (SX6)

| Parâmetro | Tipo | Descrição | Padrão |
|--|--|--|--|
| MV_SOCURL | C | URL base da API do SOC | (vazio) |
| MV_SOCTOKN | C | Token de autenticação | (vazio) |
| MV_SOCEMP | C | Código da empresa no SOC | (vazio) |
| MV_SOCTMO | N | Timeout HTTP em segundos | 30 |
| MV_SOCPATH | C | Prefixo dos endpoints | /api/v1 |

## Tabela De/Para (ZSC)

Campos: ZSC_TIPO (FIL, CC, CARGO, TURNO, SITUACAO, MOTRES), ZSC_PROT (código Protheus), ZSC_SOC (código SOC). Índice 1: ZSC_FILIAL+ZSC_TIPO+ZSC_PROT. Sem registro, usa o código Protheus.

## Retorno padronizado (JsonObject)

| Propriedade | Tipo | Descrição |
|--|--|--|
| success | logical | .T. sucesso / .F. falha |
| message | character | Mensagem descritiva |
| id | character | ID/código devolvido pelo SOC |
| http | numeric | Status HTTP (0 = falha local) |
| data / raw | object / character | Resposta do SOC (sucesso) / corpo bruto (falha) |

## Tratamento de status HTTP

| Status | Tratamento |
|--|--|
| 200/201 | Sucesso; extrai id/codigo da resposta |
| 400 | Rejeição de regra de negócio; não reenviar sem corrigir |
| 401/403 | Falha de autenticação/autorização |
| >= 500 | Indisponibilidade SOC; candidato a retry |

## Ordem recomendada de envio

Filial > Setor > Cargo > Turno > Funcionário > Desligamento.

## Exemplo

```advpl
using namespace custom.integration.soc
local oRet := SocFuncionario():new():incluir(xFilial("SRA") + "000123")
if !oRet['success']
    ConOut(oRet['message'])
endif
```

## Pendências / validações

- Paths e nomes de campos JSON são parametrizáveis e devem ser conferidos na documentação do SOC Integrador do tenant.
- Validar na release do ambiente: FWRest (GetHTTPCode, Put, SetPostParams) e FwLogMsg.
- Centro de custo: em releases atuais pode ser a CTT; ajustar SocSetor se necessário.
- Testar em homologação antes de produção.
