# Dicionário de dados — Panorama Consórcio

Resumo do modelo semântico das duas versões (`DataBase_Consorcios` = oficial, `DataBase_Consorcios_v2` = proposta com dados do BCB). A v2 é um superconjunto da oficial: tudo que existe na oficial existe na v2.

A página **Documentação** do próprio relatório traz este mesmo dicionário gerado a partir do modelo, e é a fonte que acompanha automaticamente qualquer mudança de fórmula.

## Tabelas

### `ft_Consorcios` — fato operacional (oficial e v2)

Fonte: `Base/Consolidado.xlsx`, aba `SegmentosExtraidos` (base mensal ABAC, 73.536 linhas). Uma linha por administradora × segmento × mês.

**Chaves e atributos:** `#Nome_da_Administradora`, `CNPJ_da_Administradora`, `Código_do_segmento`, `Data Base`, `Taxa_de_administração`

**Estoques de cotas ativas:** `Quantidade_de_cotas_ativas_em_dia`, `Quantidade_de_cotas_ativas_contempladas_inadimplentes`, `Quantidade_de_cotas_ativas_não_contempladas_inadimplentes`, `Quantidade_acumulada_de_cotas_ativas_contempladas`, `Quantidade_de_cotas_ativas_não_contempladas`, `Quantidade_de_cotas_ativas_quitadas`, `Quantidade_de_cotas_ativas_com_crédito_pendente_de_utilização`

**Fluxos mensais:** `Quantidade_de_cotas_comercializadas_no_mês`, `Quantidade_de_cotas_ativas_contempladas_no_mês`, `Quantidade_de_grupos_constituídos_no_mês`, `Quantidade_de_grupos_encerrados_no_mês`

**Grupos e cotas fora da carteira:** `Quantidade_de_grupos_ativos`, `Quantidade_de_cotas_excluídas`, `Quantidade_de_cotas_excluídas_a_comercializar`

> A consulta Power Query descartava seis dessas colunas (excluídas, excluídas a comercializar, grupos constituídos, grupos encerrados, quitadas e crédito pendente). Elas foram reativadas — o passo `Colunas Removidas` continua na consulta, mas com lista vazia.

### `ft_BCB_Financeiro` — fato financeiro (só v2)

Fonte: `Base/BCB/202603CONSORCIOS.CSV`, filtrado para o documento 4010 e pivotado. Uma linha por administradora × competência.

`CNPJ`, `Administradora`, `Competencia`, `Patrimonio_Liquido`, `Total_Ativo`, `Ativo_Realizavel`, `Ativo_Total_Calc`, `Total_Passivo`, `Disponibilidades`, `TVM`, `Receitas_Operacionais`, `Despesas_Operacionais`, `Receita_Taxa_Administracao`, `Resultado_Periodo`, `Margem_Resultado`, `Liquidez_Disp_TVM`, `PL_sobre_Ativo`

### `ft_BCB_ContaDetalhe` — detalhe de contas (só v2)

`CNPJ`, `Grupo`, `Ordem`, `Conta`, `Saldo`.

### Dimensões

- **`dAdministradora`** — calculada a partir de `ft_Consorcios`. Uma linha por CNPJ, com o nome mais recente e a flag `Proibida` (identifica "PROIB" no nome histórico, sinalizando restrição regulatória).
- **`dSegmento`** — fixa: Imóveis, Pesados, Motocicletas, Automóveis, Serviços, Eletroeletrônicos, cada um com ordem de exibição.
- **`dCalendario`** — calendário diário para inteligência de tempo.
- **`dAdministradoraChart`** — apoio ao eixo desacoplado do gráfico de inadimplência por administradora, para o slicer de seleção não filtrar as demais barras.
- **`TopNFinanceiro`, `LimiarConcentracao`, `FaixaConcentracao`** *(v2)* — parâmetros dos visuais da página Financeiro.

## Relacionamentos

| De | Para | Chave |
|---|---|---|
| `ft_Consorcios[Código_do_segmento]` | `dSegmento[CodSegmento]` | Código do segmento |
| `ft_Consorcios[Data Base]` | `dCalendario[Data]` | Data |
| `ft_Consorcios[CNPJ_da_Administradora]` | `dAdministradora[CNPJ]` | CNPJ |
| `ft_BCB_Financeiro[CNPJ]` *(v2)* | `dAdministradora[CNPJ]` | CNPJ |
| `ft_BCB_ContaDetalhe[CNPJ]` *(v2)* | `dAdministradora[CNPJ]` | CNPJ |

## Medidas por grupo

| Pasta | v2 | Conteúdo |
|---|---|---|
| `01 Base` | 8 | Blocos elementares: cotas em dia, inadimplentes, carteira ativa, % inadimplência, vendas, grupos, porte |
| `01 Base\Fim de Período` | 2 | Versões travadas no último mês do recorte |
| `03 Risco` (+ `\Fim de Período`) | 2 | Indicadores de risco |
| `04 Taxa` | 1 | Taxa de administração média da carteira |
| `08 Atual (sem filtro)` | 7 | Ignoram o filtro de calendário e travam no último mês da base |
| `12 Inadimplência\Administradora` | 5 | Comparação contra o mercado, elegibilidade e farol de cores |
| `12 Inadimplência\Composição` | 2 | Peso de contemplados e não contemplados na inadimplência |
| `13 Grupos` | 5 | Constituídos, encerrados, saldo, ativos e cotas por grupo |
| `14 Contemplação` | 4 | Taxa mensal, espera implícita e índice ajustado por mix |
| `15 Cancelamento` | 3 | Estoque, fluxo e cancelamentos por 100 vendidas |
| `16 Ciclo da Cota` | 4 | Quitadas, contempladas e crédito pendente de utilização |
| `21 Dinâmico` | 9 | Respondem a seleção de período e filtro dinâmico |
| `23 Financeiro (BCB)` (+ `\Por Administradora`) | 24 | Indicadores patrimoniais do balancete do BCB |
| `30 HTML` | 2 | Geram o HTML das páginas Panorama e Documentação |

### Medidas-base

- **Cotas Inadimplentes** — contempladas inadimplentes + não contempladas inadimplentes. **Não inclui cotas canceladas**, que são um estoque separado.
- **Carteira Ativa** = Cotas Em Dia + Cotas Inadimplentes
- **% Inadimplência** = Cotas Inadimplentes ÷ Carteira Ativa. É a taxa da carteira inteira, não só dos contemplados.
- **Cotas Comercializadas** — indicador central de vendas.
- **Porte (Carteira Ativa)** — Pequena (<10 mil cotas), Média (10 mil a 100 mil), Grande (>100 mil). Faixas de referência iniciais, ajustáveis.

### Contemplação (`14 Contemplação`)

- **Taxa Contemplação Mensal %** — fração da fila contemplada a cada mês, em média no período.
- **Espera Implícita (meses)** = 1 ÷ taxa mensal. Projeção da taxa observada, não prazo prometido.
- **Taxa Contemplação Esperada %** — taxa que a administradora teria se cada segmento dela rodasse na média do mercado.
- **Índice de Contemplação (vs mix)** = observada ÷ esperada. Acima de 1 contempla mais rápido que o mercado no mesmo tipo de bem. **É a comparação justa entre administradoras**: a dispersão bruta chega a 10,5×, mas cai para a faixa de 0,4× a 1,9× quando se controla pelo mix de segmento — ou seja, metade da diferença aparente é produto, não desempenho.

### Cancelamento (`15 Cancelamento`)

- **Cotas Canceladas (Acum)** — estoque acumulado no fim do período. Não deve ser somado entre meses.
- **Cotas Canceladas no Período** — variação do estoque dentro do recorte.
- **Canceladas por 100 Vendidas** — lê a qualidade da venda.

> **Ressalva de qualidade (importante):** o acumulado de cotas excluídas sofre retificações na fonte. 18,7% das variações mensais por administradora são negativas e 159 das 190 administradoras têm ao menos uma queda no acumulado, o que produz valores impossíveis em nível individual. No agregado do mercado o comportamento é estável (7 quedas em 88 meses). **Use estas medidas para mercado e segmento, não para ranquear administradoras.**

### Ciclo da cota (`16 Ciclo da Cota`)

- **Cotas Quitadas (Fim)**, **Contempladas Acum (Fim)**, **Crédito Pendente (Fim)**, **% Crédito Pendente**.

O crédito pendente merece leitura cuidadosa: um percentual alto pode refletir tempo estrutural de uso (em Imóveis o crédito leva cerca de 18 meses para ser deployado), fricção na liberação pela administradora, aceleração recente da contemplação, ou mudança de comportamento do adquirente. **A base registra que a carta não foi usada, não o motivo** — não é seguro atribuir o número a nenhuma dessas causas isoladamente.

### Financeiro BCB (`23 Financeiro (BCB)`, só v2)

PL, receita e despesa operacional, resultado e margem do período, receita de taxa de administração, ativo total, PL sobre ativo, liquidez, quantidade e percentual de administradoras com prejuízo, receita por cota administrada (cruzando BCB com ABAC), concentração de mercado e flags de ranking Top N.

## Fontes

| Fonte | Arquivo | Periodicidade | Cobertura |
|---|---|---|---|
| ABAC | `Base/Consolidado.xlsx` | Mensal | jan/2019 a mai/2026 — base operacional oficial do relatório |
| Banco Central, documento 4010 | `Base/BCB/202603CONSORCIOS.CSV` | Mensal (portal mantém histórico desde 10/2025) | Balancete patrimonial — usado apenas na proposta v2 |
