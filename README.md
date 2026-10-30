# advpl

Repositório de fontes TOTVS Protheus (AdvPL/TLPP).

## Estrutura

- `Fontes/` — código-fonte (`.prw`, `.prx`, `.tlpp`, `.ch`).
- `.agents/skills/advpl-tlpp/` — skills de IA da TOTVS (geração, revisão, refatoração, testes).
- `AGENTS.md` / `CLAUDE.md` — convenções e regras para o agente ADVPL.

## Fluxo de trabalho

1. Informar o nome do fonte e o que deve ser feito (ex.: `MT410INC.prw: validar X antes de gravar Y`).
2. O agente cria uma branch, aplica o ajuste seguindo o `AGENTS.md` e abre um Pull Request.
3. O PR é revisado e aprovado manualmente.
4. Compilação e testes em homologação antes do merge.

## Observações

- Codificação dos fontes: CP-1252 (Windows-1252).
- Idioma de commits, comentários e revisões: português do Brasil.
