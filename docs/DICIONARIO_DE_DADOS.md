# Dicionário de dados — Panorama Consórcio

Resumo do modelo semântico das duas versões do projeto (`DataBase_Consorcios` = oficial, `DataBase_Consorcios_v2` = proposta com dados do BCB). A v2 é um superconjunto da versão oficial: todas as tabelas e medidas abaixo marcadas como "oficial" também existem na v2.

## Tabelas

### `ft_Consorcios` (fato operacional — oficial e v2)
Fonte: `Base/Consolidado.xlsx` (base mensal ABAC). Uma linha por administradora × segmento × mês. Colunas principais:

- `#Nome_da_Administradora`, `CNPJ_da_Administradora`, `Código_do_segmento`, `Data Base`
- `Taxa_de_administração`
- `Quantidade_de_grupos_ativos`
- `Quantidade_de_cotas_comercializadas_no_mês`
- `Quantidade_acumulada_de_cotas_ativas_contempladas`
- `Quantidade_de_cotas_ativas_não_contempladas`
- `Quantidade_de_cotas_ativas_contempladas_no_mês`
- `Quantidade_de_cotas_ativas_em_dia`
- `Quantidade_de_cotas_ativas_contempladas_inadimplentes`
- `Quantidade_de_cotas_ativas_não_contempladas_inadimplentes`

### `ft_BCB_Financeiro` (fato financeiro — só v2)
Fonte: `Base/BCB/202603CONSORCIOS.CSV`, filtrado para o documento 4010 (balanço próprio da administradora) e pivotado nas contas relevantes. Uma linha por administradora × competência.

- `CNPJ`, `Administradora`, `Competencia`
- `Patrimonio_Liquido`, `Total_Ativo`, `Ativo_Realizavel`, `Ativo_Total_Calc`, `Total_Passivo`
- `Disponibilidades`, `TVM` (títulos e valores mobiliários)
- `Receitas_Operacionais`, `Despesas_Operacionais`, `Receita_Taxa_Administracao`
- `Resultado_Periodo`, `Margem_Resultado`
- `Liquidez_Disp_TVM`, `PL_sobre_Ativo`

### `ft_BCB_ContaDetalhe` (detalhe de contas — só v2)
Detalhamento por conta contábil do mesmo balancete BCB, usado nos visuais de detalhamento da página Financeiro. Colunas: `CNPJ`, `Grupo`, `Ordem`, `Conta`, `Saldo`.

### `dAdministradora` (dimensão calculada)
Gerada a partir de `ft_Consorcios` — uma linha por CNPJ, com o nome mais recente da administradora (`Administradora`) e uma flag `Proibida` (identifica administradoras com "PROIB" no nome histórico, sinalizando restrição regulatória).

### `dSegmento` (dimensão fixa)
6 segmentos cadastrados manualmente: Imóveis, Pesados, Motocicletas, Automóveis, Serviços, Eletroeletrônicos — cada um com uma ordem de exibição própria (`Ordem`).

### `dCalendario`
Calendário padrão para inteligência de tempo (ano, mês, dia útil, etc.), relacionado a `ft_Consorcios` pela coluna `Data Base`.

### `Medidas`
Tabela vazia dedicada a hospedar as medidas DAX do modelo, organizadas por pasta (`displayFolder`). 55 medidas na versão oficial, 79 na v2.

### Parâmetros do simulador
`PesoSolidez`, `PesoSaude`, `PesoContempl`, `PesoCusto`, `dAdministradoraChart` — tabelas de apoio ao simulador de score de administradoras (página "Simulador").

### Parâmetros da página Financeiro BCB (só v2)
`TopNFinanceiro` (quantidade de administradoras no ranking Top N), `LimiarConcentracao` (limiar % para destaque de concentração), `FaixaConcentracao` (faixas Top N / Demais usadas nos visuais).

## Relacionamentos

| De | Para | Chave |
|---|---|---|
| `ft_Consorcios[Código_do_segmento]` | `dSegmento[CodSegmento]` | Código do segmento |
| `ft_Consorcios[Data Base]` | `dCalendario[Data]` | Data |
| `ft_Consorcios[CNPJ_da_Administradora]` | `dAdministradora[CNPJ]` | CNPJ |
| `ft_BCB_Financeiro[CNPJ]` *(v2)* | `dAdministradora[CNPJ]` | CNPJ |
| `ft_BCB_ContaDetalhe[CNPJ]` *(v2)* | `dAdministradora[CNPJ]` | CNPJ |

O CNPJ raiz (8 dígitos) é a chave de junção entre a base operacional (ABAC) e a base financeira (BCB) — validado por amostragem no desenvolvimento da proposta v2 (ex.: Honda 45441789, Bradesco 52568821).

## Medidas — grupos principais (`displayFolder`)

| Pasta | Nº de medidas (v2) | Conteúdo |
|---|---|---|
| `23 Financeiro (BCB)` + `\Por Administradora` | 24 | Indicadores financeiros do BCB — ver lista abaixo |
| `22 Simulador` | 13 | Score de administradoras (solidez, saúde, contemplação, custo) |
| `21 Dinâmico` | 9 | Medidas que respondem a seleção de período/filtro dinâmico |
| `10 Consumidor\Indicadores` + `\Pilares` | 10 | Indicadores voltados ao consumidor final |
| `01 Base` (+ `\Fim de Período`) | 8 | Medidas-base: cotas em dia, inadimplentes, carteira ativa, % inadimplência |
| `12 Inadimplência\Administradora` | 5 | Inadimplência por administradora |
| `08 Atual (sem filtro)` | 3 | Snapshot da última competência, ignorando filtros de período |
| `11 Perfis` | 2 | Perfis de administradora |
| `03 Risco` (+ `\Fim de Período`) | 2 | Indicadores de risco |
| `04 Taxa` | 1 | Taxa de administração |
| `30 HTML` | 2 | Medidas auxiliares para visuais HTML/texto formatado |

### Medidas-base do modelo operacional (pasta `01 Base`)

- **Cotas Em Dia** = `SUM(ft_Consorcios[Quantidade_de_cotas_ativas_em_dia])`
- **Cotas Inadimplentes** — soma das cotas ativas inadimplentes (contempladas e não contempladas)
- **Carteira Ativa** = Cotas Em Dia + Cotas Inadimplentes — a medida-mãe da inadimplência operacional; várias outras dependem dela
- **% Inadimplência** = DIVIDE(Cotas Inadimplentes, Carteira Ativa)
- **Cotas Comercializadas** = `SUM(ft_Consorcios[Quantidade_de_cotas_comercializadas_no_mês])` — indicador central de vendas
- **Grupos Ativos** = `SUM(ft_Consorcios[Quantidade_de_grupos_ativos])`
- Variantes "(Fim)" e "(Atual)" replicam essas medidas fixando a última data disponível, para uso em cards que não devem variar com o filtro de período selecionado.

### Medidas financeiras — pasta `23 Financeiro (BCB)` (só v2)

- **PL (BCB)** = `SUM(ft_BCB_Financeiro[Patrimonio_Liquido])`
- **Receita Operacional (BCB)** / **Despesa Operacional (BCB)** — somas diretas do balancete
- **Resultado do Período (BCB)** = Receita Operacional + Despesa Operacional (despesa já vem negativa na fonte)
- **Margem do Resultado (BCB)** = DIVIDE(Resultado do Período, Receita Operacional)
- **Receita Taxa Administração (BCB)** = `SUM(ft_BCB_Financeiro[Receita_Taxa_Administracao])`
- **Ativo Total (BCB)** / **PL sobre Ativo (BCB)** — indicador de capitalização
- **Liquidez Disponível (BCB)** = `SUM(ft_BCB_Financeiro[Liquidez_Disp_TVM])`
- **Qtd Administradoras com Prejuízo (BCB)** / **% Administradoras com Prejuízo (BCB)** — contagem e proporção de administradoras com resultado negativo no período
- **Receita por Cota (BCB)** = DIVIDE(Receita Taxa Administração, Carteira Ativa) — cruza dado financeiro (BCB) com dado operacional (ABAC), calculando pela primeira vez uma receita média por cota sob gestão
- **Top 10 PL (BCB)** / **Top 10 Receita Taxa Adm (BCB)** — flags de ranking via `RANKX`, usadas para destacar as maiores administradoras nos visuais
- **Resumo Concentração (BCB)**, **% Administradoras no Limiar (BCB)**, **Receita por Faixa (BCB)** — suportam o indicador de concentração de mercado (participação das maiores administradoras)
- Medidas `Cor Resultado (BCB)` e `Cor Faixa Concentração (BCB)` retornam códigos de cor condicionais para formatação visual (verde/vermelho conforme resultado, paleta por faixa Top N/Demais)

## Fontes de dados

| Fonte | Arquivo | Atualização | Cobertura |
|---|---|---|---|
| ABAC (associação das administradoras) | `Base/Consolidado.xlsx` | Mensal | Base operacional oficial do relatório — cotas, grupos, contemplações, inadimplência |
| Banco Central do Brasil, documento 4010 | `Base/BCB/202603CONSORCIOS.CSV` | Mensal (portal do BCB mantém histórico desde 10/2025) | Balancete patrimonial das administradoras — usado apenas na proposta v2 |
