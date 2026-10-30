# AdvPL/TLPP Development Guidelines

## Language and Ecosystem

This workspace contains source code for **TOTVS Protheus ERP** in the **AdvPL** (`.prw`, `.prg`, `.prx`) and **TLPP** (`.tlpp`) languages, as well as automated tests in **Python** (TIR - Interface and e2e tests).

> **Default language**: All interactions, documentation, comments, commit messages, and reviews must be in **Brazilian Portuguese**, except for code identifiers (variable names, functions, classes) that follow technical conventions in English/Protheus abbreviations.

---

## Project Structure

```
Fontes/                      → Main source code of this repository
  └── *.prw, *.tlpp          → Sources
.agents/skills/              → AI agent skills for code generation, refactoring, documentation, tests, etc.
```

---

## Code Conventions

### Naming

- **Mandatory Hungarian notation**: `c` (character), `n` (numeric), `l` (logical), `a` (array), `o` (object), `d` (date), `b` (codeblock), `x` (variant), `j` (json)
- **Source file names**: Module prefix (4 chars) + number (3 digits) — e.g.: `MATA010`, `FINA138`, `ATFA002`
- **Table fields**: Table prefix (2-3 chars) + `_` + name — e.g.: `A1_COD`, `E1_FILIAL`
- **Tables**: Alias (2-3 chars) — e.g.: `SA1` (customers), `SE1` (accounts receivable), `SD1` (purchase invoice items)
- **Multilingual constants**: `STR0001` to `STR9999` defined in `.ch`

### Type System

#### TLPP Variable Declaration Syntax

```tlpp
// CORRECT: assignment BEFORE type annotation
Local cName   := "" as Character
Local nTotal  := 0  as Numeric
Local lFound  := .F. as Logical
Local aItems  := {} as Array
Local oModel  as Object          // without assignment is also valid

// WRONG: type annotation BEFORE assignment
Local cName as Character := ""   // does not compile
```

Every function or method **must** follow this order, without exception:

```advpl
User Function MyFunction()     // 1. header
    Local  cVar := ""          // 2. Local  variables (all here)
    Private cPriv := ""        // 3. Private variables (all here)
    // executable code          // 4. logic
Return xValue                  // 5. Return
```

- **All** `Local` declarations come before any executable line of code.
- **All** `Private` declarations come right after the `Local` ones, still before the code.
- Never declare `Local` or `Private` in the middle of the flow (inside `If`, `For`, after a function call, etc.).

### Include Files

- **Modern standard:** `#include "totvs.ch"` — consolidates all framework definitions for Protheus development
- **For TLPP files:** Add `#include "tlpp-core.th"` as the **first include**, then `#include "totvs.ch"`
- **NEVER use legacy obsolete includes** (`protheus.ch`, `Ap5Mail.ch`, `ApWizard.ch`, `FileIO.ch`, `Font.ch`, `ParmType.ch`, `RWMake.ch`) — all replaced by `totvs.ch`

### Mandatory Standards

- Always filter `D_E_L_E_T_ = ' '` in SQL queries (Protheus soft-delete)
- Always filter by branch: `XX_FILIAL = xFilial('XXX')` or equivalent
- Use `FWExecStatement` or `ChangeQuery()` for SQL injection prevention
- Use `RetSqlName()` to get the physical table name (e.g.: `SA1010`)
- **Never use `%nolock%`** (or any lock hint) in embedded SQL: it is not a BeginSQL/DBAccess macro, reaches the database literally and fails at runtime (e.g. SQL Server error 102, `Incorrect syntax near '%'`). Use `FROM %table:XXX% XXX`
- Never call `GetMV()`, `SuperGetMV()`, `ExistBlock()`, `Type()`, or `Pergunte()` inside loops
- **`Function` is forbidden**: Customizations **MUST NOT** use `Function` (public scope) — **ALWAYS** use `User Function` (prefix `U_`) or `Static Function` (file-private).
- **Entry Points — `U_` prefix forbidden in the function name**: The declared name must match **exactly** the EP name defined by the standard routine (e.g., `User Function MT410INC()`, **never** `User Function U_MT410INC()`).
- **Entry Points — file name must match the EP name (mandatory)**: e.g. `MT410INC.tlpp`, `FA080BUT.tlpp`, `A010TOK.prw`. **DO NOT** use namespaces or prefixes/suffixes in the file name.
- **File encoding**: All AdvPL/TLPP source files (`.prw`, `.prg`, `.prx`, `.tlpp`, `.ch`, `.aph`) **MUST** use **CP-1252 (Windows-1252)** encoding — **NEVER** UTF-8 or any other encoding.
- **Object destruction**: ALWAYS use the class's `Destroy()` method when available. **NEVER** use `FwFreeObj()`, `FreeObj()`, or `FwFreeArray()` as a substitute when the class provides `Destroy()`.
- **`IIF` is forbidden**: Never use `IIF()` or `IF()` inline expressions — always use explicit `If/Else/EndIf` blocks (SonarQube CA4000)
- **No UI inside transactions**: Never call `MsgAlert()`, `MsgYesNo()`, `MsgInfo()`, `Aviso()`, `Help()`, `Pergunte()`, or `ParamBox()` inside `Begin Transaction / End Transaction` blocks (SonarQube CA1002)
- **No ISAM drivers**: Never use `MSCREATE()`, `DBCREATE()`, or `CRIATRAB()` — use `FWTemporaryTable` instead (SonarQube CA1000)
- **No console output**: Never use `ConOut()`, `OutErr()`, or `?` for logging — always use `FWLogMsg()` (SonarQube CA1004)
- **`cFilial` is forbidden**: Never use the variable `cFilial` directly — it is a reserved system variable. Use variations like `cFilAux`, `cFilBkp`, or obtain the branch via `xFilial('XXX')` or `FWxFilial('XXX')`
- **REST API consumption MUST use `FWRest`**: NEVER use legacy `HTTPCGet()`, `HTTPCPost()`, or `HTTPQuote()` for new code. Secrets MUST be read from `GetMV()` parameters — never hardcoded. See the `fwrest-client-generator` skill.

### Modern TLPP (preferred for new code)

- Use namespaces, type annotations, Try-Catch
- `.tlpp` extension mandatory with `#include "tlpp-core.th"` or exclusive TLPP features
- REST via annotations: `@Get`, `@Post`, `@Put`, `@Patch`, `@Delete`
- In `.tlpp` files, always declare `using namespace` at the top of the file. **NEVER** use fully qualified namespace prefixes before each class.

---

## SonarQube AdvPL/TLPP Compliance

**All** implementation, refactoring, code review, optimization, and improvement activities **MUST** comply with the SonarQube rules for AdvPL/TLPP.

> **Full rules reference**: read [.agents/skills/advpl-tlpp/references/sonarqube-rules-reference.md](.agents/skills/advpl-tlpp/references/sonarqube-rules-reference.md) before generating or reviewing code.

---

## API Symbol Validation (Mandatory)

Before generating, migrating, or refactoring code, the agent MUST validate that **every class, method, function, namespace, and parameter signature referenced in the output truly exists** in the target framework/version. Symbols must NEVER be inferred from internal model knowledge. Lookup each symbol in the official TOTVS documentation (TDN), in the codebase, or in skill references. Reject symbols not found in any of them and document the gap instead of inventing a call.

---

## Completeness Verification (Mandatory)

After any **code generation, migration, or refactoring** task, the agent MUST present a gap analysis comparing item by item the original code (or requirements) with the generated code, marking each feature as ✅ migrated, ⚠️ not-migratable (with justification), or 🔄 preserved in legacy layer. No behavior may be silently omitted.

---

## Available Agent Skills

See [.agents/skills/advpl-tlpp/references/advpl-tlpp-skills-reference.md](.agents/skills/advpl-tlpp/references/advpl-tlpp-skills-reference.md) for the full catalog.

---

## ProtheusDOC Documentation

All new code must include a `/*/{Protheus.doc}` block with at minimum: `@type`, `@author`, `@since`, `@param` (if applicable), `@return` (if applicable).

---

## Fallback Escalation Chain

1. Codebase
2. Skill reference files
3. AGENTS.md / CLAUDE.md rules
4. Official TOTVS documentation (TDN — tdn.totvs.com)
5. Ask the user for clarification — NEVER guess or fabricate
