# README — Análise de Profissionais em Vigilância Sanitária (CNES)

## O que este script faz

Conta o número de profissionais de saúde vinculados a estabelecimentos de **Vigilância Sanitária** cadastrados no SCNES, filtrando especificamente pelo:

- **Serviço:** `141` — Vigilância em Saúde
- **Classificação:** `002` — Vigilância Sanitária
- **Esfera:** Estadual e Municipal (Central de Gestão de Saúde)

Gera dois relatórios em Excel:
- `relatorio_visa_por_estado.xlsx` — total de profissionais por estado + total Brasil
- `relatorio_visa_por_municipio.xlsx` — total de profissionais por município + total Brasil

---

## Arquivos necessários

Baixe a base de dados do CNES em:
> https://cnes.datasus.gov.br/pages/downloads/arquivosBaseDados.jsp

Coloque todos os arquivos abaixo na **mesma pasta** que o script, com a competência desejada no nome (ex: `202603` = março de 2026):

| Arquivo | Para que serve |
|---|---|
| `rlEstabServClass202603.csv` | Filtrar estabelecimentos com serviço 141/002 |
| `tbEstabelecimento202603.csv` | Pegar município e estado de cada unidade |
| `tbCargaHorariaSus202603.csv` | Listar profissionais vinculados por estabelecimento |
| `tbDadosProfissionalSus202603.csv` | Identificar profissionais únicos (evitar duplicatas) |
| `tbMunicipio202603.csv` | Traduzir código IBGE para nome do município |
| `tbEstado202603.csv` | Traduzir código de UF para nome do estado |

---

## Como usar

### 1. Instalar dependências

```bash
pip install pandas openpyxl
```

### 2. Ajustar a competência (se necessário)

Abra o script `analise_visa_cnes.py` e altere a variável `COMP` na linha indicada:

```python
COMP = "202603"  # formato: AAAAMM
```

### 3. Executar

```bash
python analise_visa_cnes.py
```

---

## Como funciona por dentro

```
rlEstabServClass  →  filtra CO_SERVICO == 141 e CO_CLASSIFICACAO == 002
        ↓
        CO_UNIDADE dos estabelecimentos VISA
        ↓
tbCargaHorariaSus  →  todos os vínculos profissionais nesses estabelecimentos
        ↓
        CO_PROFISSIONAL_SUS (identificador do profissional)
        ↓
tbEstabelecimento  →  junta município e estado de cada unidade
        ↓
tbMunicipio + tbEstado  →  junta nomes legíveis
        ↓
.nunique() por estado / por município  →  conta profissionais ÚNICOS
```

### Por que usar `.nunique()` em vez de `.count()`?

Um mesmo profissional pode ter vínculo em **mais de um estabelecimento** de Vigilância Sanitária. Se usarmos `.count()`, ele seria contado duas vezes. O `.nunique()` garante que cada `CO_PROFISSIONAL_SUS` seja contado **uma única vez dentro do mesmo nível geográfico** (estado ou município).

> Nota: se um profissional atua em municípios diferentes, ele aparece nos dois municípios — o que é correto, pois ele realmente atua nos dois locais.

---

## Estrutura dos relatórios gerados

### `relatorio_visa_por_estado.xlsx`

| UF | Estado | Qtd_Profissionais |
|---|---|---|
| BR | TOTAL BRASIL | _total_ |
| AC | Acre | _n_ |
| AL | Alagoas | _n_ |
| ... | ... | ... |

### `relatorio_visa_por_municipio.xlsx`

| Cod_IBGE | Município | UF | Estado | Qtd_Profissionais |
|---|---|---|---|---|
| — | TOTAL BRASIL | BR | — | _total_ |
| 120001 | ACRELÂNDIA | AC | Acre | _n_ |
| ... | ... | ... | ... | ... |

---

## Fonte dos dados

- **Sistema:** SCNES — Sistema de Cadastro Nacional de Estabelecimentos de Saúde
- **Gestor:** DATASUS / Ministério da Saúde
- **Acesso:** https://cnes.datasus.gov.br
- **Competência testada:** Março/2026 (`202603`)
