# Integração Protheus x SOC (TLPP)

Namespace: `custom.integration.soc`

## Parâmetros SX6
MV_SOCURL, MV_SOCTOKN, MV_SOCEMP, MV_SOCTMO, MV_SOCPATH

## Tabela De/Para (ZSC)
ZSC_TIPO (FIL, CC, CARGO, TURNO, SITUACAO, MOTRES), ZSC_PROT, ZSC_SOC. Índice 1: ZSC_FILIAL+ZSC_TIPO+ZSC_PROT.

## Atenção
Paths e campos JSON são placeholders: validar com a documentação do SOC Integrador do tenant. Validar também FWRest (GetHTTPCode, Put) na release em uso.

## Ordem de envio
Filial > Setor > Cargo > Turno > Funcionário > Desligamento.
