# Panorama Consórcio — Comitê ABEFIN

Painel de inteligência de mercado do segmento de consórcios, construído em Power BI a partir de duas fontes de dados públicas/oficiais: a base mensal da ABAC (operacional — cotas, grupos, contemplações, inadimplência) e, na versão proposta (v2), o balancete mensal do Banco Central do Brasil (financeiro — patrimônio, receita, resultado das administradoras).

O projeto é mantido para o Comitê dos Profissionais de Consórcio (ABEFIN) e serve como base analítica para acompanhar tendências de vendas, inadimplência, concentração de mercado e saúde financeira das administradoras de consórcio no Brasil.

## Estrutura do repositório

```
DataBase_Consorcios.pbip                  → projeto oficial em uso (Power BI Project)
DataBase_Consorcios.Report/               → definição do relatório (páginas, visuais) da versão oficial
DataBase_Consorcios.SemanticModel/        → modelo semântico (tabelas, medidas DAX, relacionamentos) da versão oficial

DataBase_Consorcios_v2.pbip                → cópia de trabalho com a proposta da página financeira (BCB)
DataBase_Consorcios_v2.Report/             → relatório da v2, com a 5ª página "Financeiro (BCB)" já implementada
DataBase_Consorcios_v2.SemanticModel/      → modelo semântico da v2, com as tabelas e medidas do BCB

Base/                                      → dados-fonte
  Consolidado.xlsx                         → base operacional mensal (ABAC), fonte do modelo oficial
  BCB/202603CONSORCIOS.CSV                 → balancete mensal do Banco Central (documento 4010), competência 03/2026

docs/                                      → documentação de apoio (avaliação técnica e resumo de implementação da proposta BCB)
```

Os arquivos `.pbip` são projetos Power BI em formato texto (Power BI Project) — cada um é acompanhado das pastas `.Report` e `.SemanticModel` correspondentes, versionáveis normalmente em Git. Para abrir, use o Power BI Desktop e dê duplo-clique no `.pbip` desejado; as pastas `.Report`/`.SemanticModel` são lidas automaticamente, não precisam ser abertas separadamente.

Também existe um `DataBase_Consorcios.pbix` (o binário compilado da versão oficial) mantido no repositório por conveniência de distribuição — quem só precisa visualizar o relatório pronto pode abrir direto esse arquivo, sem precisar do Power BI Desktop em modo de projeto.

## Duas versões, dois propósitos

- **`DataBase_Consorcios` (oficial):** é o material em uso pelo Comitê. Cobre 4 páginas — Panorama, Análise, Simulador e Documentação — todas baseadas em dados operacionais (ABAC): cotas comercializadas, grupos ativos, contemplações e inadimplência de cotas, por administradora e por segmento (Imóveis, Pesados, Motocicletas, Automóveis, Serviços, Eletroeletrônicos).
- **`DataBase_Consorcios_v2` (proposta):** cópia independente e completa do projeto oficial, criada para avaliar uma 5ª página — "Financeiro (BCB)" — sem qualquer risco ao material oficial. Adiciona ao modelo dados de patrimônio líquido, receita, resultado e margem das administradoras, extraídos do balancete público do Banco Central, cruzados com a base operacional pelo CNPJ. **Ainda não substitui a versão oficial — depende de aprovação da presidência do Comitê** (ver `docs/Proposta_Nova_Pagina_BCB.md`).

## O que os dados cobrem

### Base operacional (ABAC — `Consolidado.xlsx`)
Granularidade: 1 linha por administradora × segmento × mês. Traz cotas comercializadas, grupos ativos, cotas contempladas e não contempladas, cotas em dia e cotas inadimplentes — a base de todos os indicadores de vendas e inadimplência operacional do relatório oficial.

### Base financeira (BCB — `202603CONSORCIOS.CSV`)
Balancete patrimonial das administradoras de consórcio, documento 4010, publicado mensalmente pelo Banco Central. Cobertura confirmada: das 117 administradoras ativas no modelo operacional (competência 03/2026), 110 (99,6% da carteira ativa) têm registro financeiro correspondente no BCB via CNPJ. O arquivo do BCB também revela 15 administradoras registradas no Banco Central que não aparecem hoje na base operacional monitorada pelo Comitê — um gap reportado em `docs/Proposta_Nova_Pagina_BCB.md`.

## Principais achados já documentados

- **Concentração de mercado:** as 5 maiores administradoras por Patrimônio Líquido concentram 58,5% do PL do sistema (HHI = 1.469); por receita de taxa de administração a concentração é bem menor (top 5 = 48,5%, HHI = 602) — o mercado é mais pulverizado em faturamento do que em patrimônio.
- **Saúde financeira:** nenhuma administradora opera com Patrimônio Líquido negativo, mas 21 de 124 administradoras (17%) fecharam a competência com resultado negativo.
- **Inadimplência × resultado financeiro:** correlação fraca e negativa (r ≈ −0,22) — a inadimplência de cotas explica só uma pequena parte do resultado financeiro das administradoras; estrutura de custos pesa mais.
- Detalhamento completo, ressalvas metodológicas e a proposta de conteúdo da nova página estão em `docs/Proposta_Nova_Pagina_BCB.md` e `docs/Implementacao_v2_BCB.md`.

## Modelo de dados (resumo)

| Tabela | Papel | Granularidade |
|---|---|---|
| `ft_Consorcios` | Fato operacional (ABAC) | Administradora × Segmento × Mês |
| `ft_BCB_Financeiro` *(só v2)* | Fato financeiro (BCB) | Administradora × Competência |
| `ft_BCB_ContaDetalhe` *(só v2)* | Detalhe de contas do balancete BCB | Administradora × Conta contábil |
| `dAdministradora` | Dimensão, calculada a partir do `ft_Consorcios` | 1 linha por CNPJ |
| `dSegmento` | Dimensão fixa (6 segmentos) | 1 linha por segmento |
| `dCalendario` | Calendário para inteligência de tempo | 1 linha por dia |
| `Medidas` | Tabela dedicada a medidas DAX (79 medidas na v2, 55 na versão oficial) | — |
| `PesoSolidez`, `PesoSaude`, `PesoContempl`, `PesoCusto` | Parâmetros do simulador de score de administradoras | — |
| `TopNFinanceiro`, `LimiarConcentracao`, `FaixaConcentracao` *(só v2)* | Parâmetros dos visuais de ranking/concentração da página BCB | — |

O relacionamento entre as bases operacional e financeira é feito pelo CNPJ raiz (8 dígitos) da administradora, no mesmo padrão usado em todo o modelo.

## Como abrir e atualizar

1. Abra `DataBase_Consorcios.pbip` (oficial) ou `DataBase_Consorcios_v2.pbip` (proposta) no Power BI Desktop.
2. Nos parâmetros do Power Query (`Caminho`, `Base`), confirme que o caminho aponta para a pasta `Base/` deste repositório.
3. Clique em **Atualizar** (Refresh) para recarregar os dados — os arquivos `.pbip` guardam a definição do modelo e do relatório, não os dados em si.

## Avisos legais e de uso

O material não constitui avaliação de crédito, rating ou recomendação de investimento. Os dados financeiros do Banco Central usados na proposta v2 são públicos, mas de leitura institucionalmente sensível (expõem resultado e patrimônio por administradora nomeada) — por isso essa página permanece em avaliação e não faz parte do material oficial distribuído até aprovação da presidência.

## Histórico

Este repositório foi versionado a partir da pasta de trabalho local do Comitê, consolidando o material oficial e a proposta técnica da página financeira (BCB) desenvolvida em agosto de 2026.
