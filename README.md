# Introdução a Sistemas Inteligentes — Trabalho 1

Tratamento da base de gastos com o Cartão de Pagamento do Governo Federal (CPGF) para uso nas análises e modelos do grupo.

## Estrutura

```
data/
├── 202608_CPGF.csv           # base original (Portal da Transparência)
├── base_cpgf_tratada.csv     # base final tratada (gerada pelo notebook)
└── etl/
    └── data-cleaning.ipynb   # notebook de limpeza (ETL)
requirements.txt
```

## Como rodar

```bash
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
.venv/bin/python -m ipykernel install --user --name trabalho1-venv --display-name "Python (.venv trabalho-1)"
```

Abra `data/etl/data-cleaning.ipynb`, selecione o kernel **Python (.venv trabalho-1)** e execute todas as células. A base tratada é gravada em `data/base_cpgf_tratada.csv`.

## Origem dos dados

- **Fonte:** Portal da Transparência do Governo Federal, conjunto *Cartão de Pagamento do Governo Federal (CPGF)*.
- **Arquivo:** `202608_CPGF.csv`, extrato de agosto de 2026, com transações entre 26/06/2026 e 29/07/2026.
- **Granularidade:** uma linha por transação feita com o cartão corporativo por um servidor (portador) de um órgão federal.
- **Formato original:** 15.237 linhas, 15 colunas, separador `;`, codificação `latin-1`, valores no padrão `1.234,56` e datas em `dd/mm/aaaa`. Os CPFs vêm parcialmente mascarados pelo próprio portal.

## Processo de ETL

1. **Extração:** leitura do CSV com `sep=';'`, `encoding='latin-1'` e todas as colunas como texto.
2. **Redução:** de 15 para 6 colunas. Códigos numéricos, CPF/CNPJ e ano/mês do extrato foram descartados.
3. **Valores:** remoção do ponto de milhar, troca da vírgula por ponto e conversão para `float`.
4. **Nulos:** remoção de 3.926 linhas (25,8%) do tipo *"Informações protegidas por sigilo"*, que não têm data nem CPF.
5. **Datas:** conversão de `dd/mm/aaaa` para `datetime`.
6. **Falso ausente:** nos 1.272 saques, o favorecido `"NAO SE APLICA"` foi trocado por `"SAQUE (SEM FAVORECIDO)"`.
7. **Duplicatas:** 476 linhas idênticas foram **mantidas**. O arquivo não tem ID de transação, e elas devem ser compras repetidas legítimas.
8. **Padronização:** colunas renomeadas para `snake_case`, sem acentos.
9. **Carga:** exportação para `data/base_cpgf_tratada.csv` (UTF-8, separador `,`, sem coluna de índice).

**Resultado:** 11.311 linhas e 6 colunas, sem nulos, com valores entre R$ 0,10 e R$ 23.556,00.

> **Limitação:** as transações sigilosas removidas somam R$ 4.532.498,68, ou **38,7% do valor total** do extrato (R$ 11.724.649,65). Totais de gasto calculados na base tratada, sobretudo por órgão, ficam subestimados.

## Dicionário de dados — `data/base_cpgf_tratada.csv`

| Coluna | Tipo | Coluna original | Descrição | Exemplo |
|---|---|---|---|---|
| `orgao_superior` | texto | NOME ÓRGÃO SUPERIOR | Órgão superior (ministério ou equivalente) do portador. 29 valores distintos. | Advocacia-Geral da União |
| `portador` | texto | NOME PORTADOR | Servidor titular do cartão que fez a transação. | ISABELA RODRIGUES TEIXEIRA |
| `favorecido` | texto | NOME FAVORECIDO | Estabelecimento/fornecedor que recebeu o pagamento. Em saques, `SAQUE (SEM FAVORECIDO)`. | VLB COMERCIO VAREJISTA DE FERRAGENS LTDA |
| `tipo_transacao` | texto (categoria) | TRANSAÇÃO | Tipo da operação: compra à vista ou saque, em R$ ou moeda internacional (INT$). | COMPRA A/V - R$ - APRES |
| `data` | data (`aaaa-mm-dd`) | DATA TRANSAÇÃO | Data em que a transação ocorreu. | 2026-07-13 |
| `valor` | decimal (R$) | VALOR TRANSAÇÃO | Valor da transação em reais, separador decimal `.`. | 97.49 |

### Valores de `tipo_transacao`

| Valor | Significado | Linhas |
|---|---|---|
| `COMPRA A/V - R$ - APRES` | Compra à vista em reais | 10.011 |
| `SAQUE CASH/ATM BB` | Saque em caixa eletrônico do Banco do Brasil | 1.218 |
| `SAQUE - INT$ - APRES` | Saque em moeda estrangeira | 54 |
| `COMPRA A/V - INT$ - APRES` | Compra à vista em moeda estrangeira | 27 |
| `COMP A/V-SOL DISP C/CLI-R$ ANT VENC` | Compra à vista em reais com liquidação antecipada | 1 |
