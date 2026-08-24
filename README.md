# Panorama Consórcio — Comitê ABEFIN

Painel de Power BI (formato PBIP) de inteligência de mercado sobre o setor brasileiro de consórcios, construído a partir de dados públicos ABAC/BACEN de janeiro de 2019 a maio de 2026 — 160 administradoras registradas, 124 com carteira ativa, 6 segmentos (Imóveis, Automóveis, Motocicletas, Serviços, Pesados, Eletroeletrônicos).

O projeto nasceu de uma pergunta de pesquisa própria — há vínculo entre o crescimento do consórcio e a alta da taxa de juros, e qual o impacto da inadimplência nesse período — e evoluiu para material oficial do Comitê dos Profissionais de Consórcio (ABEFIN).

## Estrutura do repositório

```
DataBase_Consorcios.pbip                  → projeto oficial em uso
DataBase_Consorcios.Report/               → relatório da versão oficial
DataBase_Consorcios.SemanticModel/        → modelo semântico da versão oficial

DataBase_Consorcios_v2.pbip                → cópia de trabalho com a proposta da página financeira (BCB)
DataBase_Consorcios_v2.Report/             → relatório da v2, com a página "Financeiro (BCB)"
DataBase_Consorcios_v2.SemanticModel/      → modelo semântico da v2

Base/
  Consolidado.xlsx                         → base operacional mensal (ABAC), aba SegmentosExtraidos, 73.536 linhas
  BCB/202603CONSORCIOS.CSV                 → balancete do Banco Central (documento 4010), competência 03/2026

docs/                                      → dicionário de dados e documentos técnicos da proposta BCB
```

Os `.pbip` são projetos Power BI em formato texto, versionáveis normalmente em Git. Para abrir, dê duplo-clique no `.pbip` no Power BI Desktop — as pastas `.Report`/`.SemanticModel` são lidas automaticamente. Há também um `DataBase_Consorcios.pbix` mantido por conveniência, para quem só precisa visualizar o relatório pronto.

## Duas versões, dois propósitos

- **`DataBase_Consorcios` (oficial)** — material em uso pelo Comitê. Três páginas: **Panorama**, **Análise** e **Documentação**, sobre dados operacionais da ABAC.
- **`DataBase_Consorcios_v2` (proposta)** — cópia independente que acrescenta a 4ª página **Financeiro (BCB)**, com patrimônio líquido, receita, resultado e margem das administradoras, cruzados com a base operacional pelo CNPJ. **Depende de aprovação da presidência** (ver `docs/Proposta_Nova_Pagina_BCB.md`).

## Páginas

| Página | Conteúdo |
|---|---|
| **Panorama** | Leitura narrativa do mercado em quatro atos: tamanho, segmentos, concentração de risco e leitura metodológica. |
| **Análise** | Tela interativa: ficha individual da administradora, comparador entre administradoras e abertura por segmento. |
| **Documentação** | Dicionário completo das medidas, gerado a partir do próprio modelo semântico. |
| **Financeiro (BCB)** *(só v2)* | KPIs patrimoniais do sistema, concentração de mercado e ranking por PL e receita. |

> **Ressalva conhecida sobre a página Panorama:** ela é HTML estático — não referencia nenhuma medida do modelo. Os números foram auditados contra a base e conferem exatamente para a competência mai/2026, mas **não acompanham atualização de dados**. É uma decisão consciente por ora; a página será repensada quando a narrativa mudar.

## Modelo de dados

| Tabela | Papel | Granularidade |
|---|---|---|
| `ft_Consorcios` | Fato operacional (ABAC) | Administradora × Segmento × Mês |
| `ft_BCB_Financeiro` *(v2)* | Fato financeiro (BCB) | Administradora × Competência |
| `ft_BCB_ContaDetalhe` *(v2)* | Detalhe de contas do balancete | Administradora × Conta contábil |
| `dAdministradora` | Dimensão calculada a partir do fato | 1 linha por CNPJ |
| `dSegmento` | Dimensão fixa | 6 segmentos |
| `dCalendario` | Calendário para inteligência de tempo | 1 linha por dia |
| `Medidas` | Tabela dedicada às medidas DAX (78 na v2, 54 na oficial) | — |
| `dAdministradoraChart` | Apoio ao eixo desacoplado do gráfico por administradora | — |
| `TopNFinanceiro`, `LimiarConcentracao`, `FaixaConcentracao` *(v2)* | Parâmetros dos visuais da página Financeiro | — |

A junção entre a base operacional e a financeira é feita pelo CNPJ raiz (8 dígitos).

### Grupos de medidas

`01 Base` · `03 Risco` · `04 Taxa` · `08 Atual (sem filtro)` · `12 Inadimplência` (Administradora e Composição) · `13 Grupos` · `14 Contemplação` · `15 Cancelamento` · `16 Ciclo da Cota` · `21 Dinâmico` · `23 Financeiro (BCB)` · `30 HTML`

Detalhamento completo em [`docs/DICIONARIO_DE_DADOS.md`](docs/DICIONARIO_DE_DADOS.md) e na página Documentação do próprio relatório.

## Como abrir e atualizar

1. Abra `DataBase_Consorcios.pbip` ou `DataBase_Consorcios_v2.pbip` no Power BI Desktop.
2. Confirme que os parâmetros do Power Query (`Caminho`, `Base`) apontam para a pasta `Base/` deste repositório.
3. Clique em **Atualizar** — os `.pbip` guardam a definição do modelo e do relatório, não os dados.

## Princípios editoriais

O material é **descritivo, não avaliativo**. Decisões metodológicas adotadas:

- **Sem linguagem de julgamento.** Não há "melhor" ou "pior" administradora, nem "nicho arriscado" ou "seguro". Os rankings são rotulados pelo que medem: maiores e menores taxas de inadimplência.
- **Volume e taxa são apresentados separados.** Uma administradora grande naturalmente concentra mais cotas em atraso; isso é aritmética de porte, não desempenho. Por isso a tabela de concentração mostra fatia da carteira e fatia da inadimplência lado a lado.
- **Comparações controladas por segmento.** O `Índice de Contemplação (vs mix)` compara cada administradora com a média do mercado nos segmentos em que ela própria atua — metade da dispersão bruta entre administradoras é mix de produto, não desempenho.
- **Faixas fixas, não relativas à média.** O farol de inadimplência usa cortes fixos (azul <10%, amarelo 10–20%, vermelho >20%), para a leitura não mudar conforme o filtro aplicado.
- **Cotas canceladas não entram na inadimplência.** A métrica usa apenas cotas ativas. Cancelamentos são acompanhados em medidas próprias.

## Avisos legais e de uso

O material não constitui avaliação de crédito, rating ou recomendação de investimento. Os dados financeiros do Banco Central usados na proposta v2 são públicos, mas de leitura institucionalmente sensível — expõem resultado e patrimônio por administradora nomeada. Por isso a página permanece em avaliação e não faz parte do material oficial até aprovação da presidência.

**Limitação de qualidade de dados conhecida:** o acumulado de cotas excluídas sofre retificações na fonte — 18,7% das variações mensais por administradora são negativas e 159 das 190 administradoras têm ao menos uma queda no acumulado. No agregado do mercado o comportamento é estável (7 quedas em 88 meses). As medidas de cancelamento servem para leitura de mercado e de segmento, **não para ranquear administradoras**.
