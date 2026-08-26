# Dicionário de dados — Panorama Consórcio

Resumo do modelo semântico. Desde que a página Financeiro (BCB) foi portada para a versão oficial, `DataBase_Consorcios` e `DataBase_Consorcios_v2` têm o mesmo modelo: 12 tabelas e 82 medidas.

A página **Documentação** do próprio relatório traz este mesmo dicionário, mantido junto ao modelo, e é a referência que acompanha o arquivo para onde ele for.

## Tabelas

### `ft_Consorcios` — fato operacional

Fonte: `Base/Consolidado.xlsx`, aba `SegmentosExtraidos` (base mensal ABAC, 73.536 linhas). Uma linha por administradora × segmento × mês.

**Chaves e atributos:** `#Nome_da_Administradora`, `CNPJ_da_Administradora`, `Código_do_segmento`, `Data Base`, `Taxa_de_administração`

**Estoques de cotas ativas:** `Quantidade_de_cotas_ativas_em_dia`, `Quantidade_de_cotas_ativas_contempladas_inadimplentes`, `Quantidade_de_cotas_ativas_não_contempladas_inadimplentes`, `Quantidade_acumulada_de_cotas_ativas_contempladas`, `Quantidade_de_cotas_ativas_não_contempladas`, `Quantidade_de_cotas_ativas_quitadas`, `Quantidade_de_cotas_ativas_com_crédito_pendente_de_utilização`

**Fluxos mensais:** `Quantidade_de_cotas_comercializadas_no_mês`, `Quantidade_de_cotas_ativas_contempladas_no_mês`, `Quantidade_de_grupos_constituídos_no_mês`, `Quantidade_de_grupos_encerrados_no_mês`

**Grupos e cotas fora da carteira:** `Quantidade_de_grupos_ativos`, `Quantidade_de_cotas_excluídas`, `Quantidade_de_cotas_excluídas_a_comercializar`

> A consulta Power Query descartava seis dessas colunas (excluídas, excluídas a comercializar, grupos constituídos, grupos encerrados, quitadas e crédito pendente). Elas foram reativadas — o passo `Colunas Removidas` continua na consulta, mas com lista vazia.

### `ft_BCB_Financeiro` — fato financeiro

Fonte: `Base/BCB/202603CONSORCIOS.CSV`, filtrado para o documento 4010 e pivotado. Uma linha por administradora × competência.

`CNPJ`, `Administradora`, `Competencia`, `Patrimonio_Liquido`, `Total_Ativo`, `Ativo_Realizavel`, `Ativo_Total_Calc`, `Total_Passivo`, `Disponibilidades`, `TVM`, `Receitas_Operacionais`, `Despesas_Operacionais`, `Receita_Taxa_Administracao`, `Resultado_Periodo`, `Margem_Resultado`, `Liquidez_Disp_TVM`, `PL_sobre_Ativo`

> O balancete é **acumulado no semestre**. Para a competência 03/2026, os valores representam três meses, não um. Toda leitura "por mês" precisa dividir pelos meses decorridos.

### `ft_BCB_ContaDetalhe` — detalhe de contas

`CNPJ`, `Grupo`, `Ordem`, `Conta`, `Saldo`. Alimenta a matriz de balanço padrão por administradora selecionada.

### Dimensões e parâmetros

- **`dAdministradora`** — calculada a partir de `ft_Consorcios`. Uma linha por CNPJ, com o nome mais recente e a flag `Proibida` (identifica "PROIB" no nome histórico, sinalizando restrição regulatória).
- **`dSegmento`** — fixa: Imóveis, Pesados, Motocicletas, Automóveis, Serviços, Eletroeletrônicos, cada um com ordem de exibição.
- **`dCalendario`** — calendário diário para inteligência de tempo.
- **`dAdministradoraChart`** — apoio ao eixo desacoplado do gráfico de inadimplência por administradora, para o slicer de seleção não filtrar as demais barras.
- **`Indicadores_`** — parâmetro de campo com 15 medidas: deixa o profissional escolher o que comparar no comparador de administradoras. **Tabela desconectada por natureza** — não crie relação para ela.
- **`TopNFinanceiro`, `LimiarConcentracao`, `FaixaConcentracao`** — parâmetros dos visuais da página Financeiro.

## Relacionamentos

| De | Para | Chave |
|---|---|---|
| `ft_Consorcios[Código_do_segmento]` | `dSegmento[CodSegmento]` | Código do segmento |
| `ft_Consorcios[Data Base]` | `dCalendario[Data]` | Data |
| `ft_Consorcios[CNPJ_da_Administradora]` | `dAdministradora[CNPJ]` | CNPJ |
| `ft_BCB_Financeiro[CNPJ]` | `dAdministradora[CNPJ]` | CNPJ |
| `ft_BCB_ContaDetalhe[CNPJ]` | `dAdministradora[CNPJ]` | CNPJ |

## Medidas por grupo

| Pasta | Medidas | Conteúdo |
|---|---|---|
| `01 Base` | 8 | Blocos elementares: cotas em dia, inadimplentes, carteira ativa, vendas, contempladas, grupos, porte |
| `01 Base\Fim de Período` | 2 | Versões travadas no último mês do recorte |
| `03 Risco` | 1 | % Inadimplência |
| `03 Risco\Fim de Período` | 1 | % Inadimplência (Fim) |
| `04 Taxa` | 1 | Taxa de administração média da carteira |
| `08 Atual (sem filtro)` | 7 | Ignoram o filtro de calendário e travam no último mês da base |
| `12 Inadimplência\Administradora` | 5 | Corte de porte, elegibilidade de ranking e farol de cores |
| `12 Inadimplência\Composição` | 3 | Peso de contemplados e não contemplados, e a variação em 12 meses |
| `13 Grupos` | 5 | Constituídos, encerrados, saldo, ativos e cotas por grupo |
| `14 Contemplação` | 3 | Taxa mensal, taxa esperada e índice ajustado por mix |
| `15 Cancelamento` | 4 | Estoque, fluxo, cancelamentos por 100 ativas e por 100 vendidas |
| `16 Ciclo da Cota` | 4 | Quitadas, contempladas e crédito pendente de utilização |
| `21 Dinâmico` | 9 | Respondem à seleção de período e ao filtro dinâmico da página Análise |
| `23 Financeiro (BCB)` | 21 | Indicadores patrimoniais do balancete do BCB |
| `23 Financeiro (BCB)\Por Administradora` | 4 | Cores condicionais e sinalizadores de Top N |
| `30 HTML` | 2 | Geram o HTML das páginas Panorama e Documentação |
| **Subtotal `Medidas`** | **80** | |
| `TopNFinanceiro`, `LimiarConcentracao` | 2 | Medidas de leitura dos parâmetros |
| **Total** | **82** | |

### Medidas-base

- **Cotas Inadimplentes** — contempladas inadimplentes + não contempladas inadimplentes. **Não inclui cotas canceladas**, que são um estoque separado.
- **Carteira Ativa** = Cotas Em Dia + Cotas Inadimplentes. É **estoque**: somar meses não faz sentido.
- **% Inadimplência** = Cotas Inadimplentes ÷ Carteira Ativa. É a taxa da carteira inteira, não só dos contemplados.
- **Cotas Comercializadas** — indicador central de vendas.
- **Porte (Carteira Ativa)** — Pequena (<10 mil cotas), Média (10 mil a 100 mil), Grande (>100 mil). Faixas de referência iniciais, ajustáveis.

### Contemplação (`14 Contemplação`)

- **Taxa Contemplação Mensal %** — fração das cotas ainda não contempladas que é contemplada a cada mês, em média no período.
- **Taxa Contemplação Esperada %** — taxa que a administradora teria se cada segmento dela rodasse na média do mercado.
- **Índice de Contemplação (vs mix)** = observada ÷ esperada. Acima de 1 contempla mais rápido que o mercado no mesmo tipo de bem. **É a comparação justa entre administradoras**: a dispersão bruta chega a 10,5×, mas cai para a faixa de 0,4× a 1,9× quando se controla pelo mix de segmento — ou seja, metade da diferença aparente é produto, não desempenho.

> A antiga medida **Espera Implícita (meses)** foi removida. Ela calculava `1 ÷ taxa mensal`, fórmula válida apenas para uma fila em regime estável; o consórcio é de prazo fechado e com fila em crescimento acelerado, o que produzia projeções de 200 a 600 meses, sem significado.

### Cancelamento (`15 Cancelamento`)

- **Cotas Canceladas (Acum)** — estoque acumulado no fim do período. Não deve ser somado entre meses.
- **Cotas Canceladas no Período** — soma apenas as variações **positivas** do estoque acumulado mês a mês. Quedas no acumulado são retificação da fonte e entram como zero.
- **Canceladas por 100 Ativas** — indicador de estoque, estável em qualquer recorte.
- **Canceladas por 100 Vendidas** — lê a qualidade da venda.

> **Ressalva de qualidade (importante):** o acumulado de cotas excluídas sofre retificações na fonte. 18,7% das variações mensais por administradora são negativas e 159 das 190 administradoras têm ao menos uma queda no acumulado, o que produz valores impossíveis em nível individual. No agregado do mercado o comportamento é estável (7 quedas em 88 meses). **Use estas medidas para mercado e segmento, não para ranquear administradoras.**

### Ciclo da cota (`16 Ciclo da Cota`)

- **Cotas Quitadas (Fim)**, **Contempladas Acum (Fim)**, **Crédito Pendente (Fim)**, **% Crédito Pendente**.

O crédito pendente merece leitura cuidadosa: um percentual alto pode refletir tempo estrutural de uso (em Imóveis o crédito leva cerca de 18 meses para ser deployado), fricção na liberação pela administradora, aceleração recente da contemplação, ou mudança de comportamento do adquirente. **A base registra que a carta não foi usada, não o motivo** — não é seguro atribuir o número a nenhuma dessas causas isoladamente.

### Financeiro BCB (`23 Financeiro (BCB)`)

PL, receita e despesa operacional, resultado e margem do período, receita de taxa de administração, ativo total, PL sobre ativo, liquidez, quantidade e percentual de administradoras com prejuízo, receita por cota administrada (cruzando BCB com ABAC), concentração de mercado e flags de ranking Top N.

> **Receita por Cota (BCB)** usa como denominador a carteira ativa **na competência do balancete**, não a soma dos meses do recorte. Carteira ativa é estoque; somá-la ao longo de 89 meses inflava o denominador em cerca de 65 vezes e derrubava o indicador para poucos reais por cota. Para leitura mensal, use `Receita por Cota ao Mês (BCB)`, que divide pelo número de meses acumulados no semestre.

> **Ativo Total e PL** não são comparáveis entre administradoras de perfis distintos: para as ligadas a conglomerados financeiros maiores, o "Total Geral do Ativo" pode refletir o balanço da instituição inteira. Para concentração de mercado, prefira `Receita Top N % (BCB)`.

## Quebras de série e limitações

- **jan/2024 — Resolução BCB nº 285.** A exclusão da cota passou a ocorrer após três meses consecutivos de inadimplência. A taxa do mercado caiu 1,55 p.p. em um único mês, contra 0,27 p.p. de variação típica. Comparações que cruzam essa data medem também a mudança de regra.
- **2026 é ano parcial.** A base vai até maio. CAGR e comparações anuais usam somente anos completos, de 2019 a 2025.
- **Não existe valor em reais na base operacional.** A ABAC traz quantidades de cotas. Valores em reais só existem no balancete do BCB, em nível de administradora, não de cota.
- **Divergência de classificação ABAC × base.** Automóveis e motocicletas, tomados individualmente, divergem entre a publicação da ABAC e esta base; a soma dos dois bate dentro de 0,3%.

## Fontes

| Fonte | Arquivo | Periodicidade | Cobertura |
|---|---|---|---|
| ABAC | `Base/Consolidado.xlsx` | Mensal | jan/2019 a mai/2026 — base operacional oficial do relatório |
| Banco Central, documento 4010 | `Base/BCB/202603CONSORCIOS.CSV` | Semestral acumulado, publicação mensal | Balancete patrimonial, competência 03/2026 |
