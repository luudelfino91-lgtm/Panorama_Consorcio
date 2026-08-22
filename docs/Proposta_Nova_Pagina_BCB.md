# Avaliação técnica e proposta de nova página — Dados patrimoniais BCB (Balancete 03/2026)

**Preparado para:** Comitê dos Profissionais de Consórcio (ABEFIN)
**Base analisada:** `202603CONSORCIOS.csv` — Balancete/Balanço Patrimonial de administradoras de consórcio, Banco Central do Brasil (documento 4010, competência 03/2026)
**Status:** Proposta técnica para avaliação da presidência — **não implementada no relatório oficial**

---

## 1. Avaliação de aderência ao material

**Conclusão: o arquivo se encaixa muito bem e adiciona uma dimensão hoje inexistente no modelo — dados financeiros reais das administradoras.**

O modelo atual (`Consolidado.xlsx`, fonte ABAC/mensal) traz apenas indicadores **operacionais**: cotas, grupos, contemplações, inadimplência de cotas. Não existe, hoje, nenhuma medida de saúde **financeira** das administradoras (patrimônio, receita, resultado). O arquivo do BCB preenche exatamente essa lacuna.

Testes de compatibilidade realizados:

- **Chave de junção:** o CNPJ (raiz, 8 dígitos) do arquivo BCB é idêntico em formato ao `CNPJ` já usado em `dAdministradora`/`ft_Consorcios`. Confirmado por amostragem (ex.: Honda 45441789, Bradesco 52568821) — join direto, sem tratamento adicional.
- **Cobertura:** das 117 administradoras ativas no modelo em 03/2026, **110 (99,6% da carteira ativa total)** têm registro financeiro correspondente no BCB. As 7 sem correspondência são entidades de porte residual (associações e fundações com carteira própria pequena — ex. Fundação Hab. do Exército, Clube Naval), sem materialidade.
- **Achado adicional:** o arquivo do BCB traz **15 administradoras que não aparecem no modelo operacional atual** (ex.: MYCON, EVOY, PRIMO ROSSI, EUTBEM, SERELLO — algumas com receita de taxa de administração relevante, casos de EVOY R$42,8 milhões e MYCON R$23,4 milhões). Isso sugere que o universo hoje monitorado pelo Comitê está ligeiramente incompleto frente ao total de administradoras registradas no BCB — um gap que vale reportar independentemente da nova página.
- **Periodicidade:** o BCB publica esse balancete mensalmente (mesmo portal lista de 10/2025 a 03/2026), o que permite atualização recorrente, no mesmo ritmo do restante do relatório.

---

## 2. Análises realizadas (base: competência 03/2026, 110 administradoras cruzadas)

### 2.1 Panorama financeiro do sistema

| Métrica (soma das 110 administradoras cruzadas) | Valor |
|---|---|
| Patrimônio Líquido total | R$ 21,2 bilhões |
| Receitas Operacionais totais | R$ 8,8 bilhões |
| Receita de Taxa de Administração | R$ 7,1 bilhões |
| Resultado aproximado do período (Receitas − Despesas) | R$ 4,2 bilhões (positivo) |

### 2.2 Concentração de mercado (base completa BCB, 125 administradoras)

- As **5 maiores administradoras por Patrimônio Líquido concentram 58,5%** de todo o PL do sistema; as 10 maiores, 75,3%. Índice HHI (PL) = **1.469** — mercado moderadamente concentrado.
- Por **Receita de Taxa de Administração**, a concentração é menor: top 5 = 48,5%, HHI = **602** — mercado mais pulverizado em faturamento do que em patrimônio.
- Ranking por PL: Bradesco (R$ 7,52 bi) muito à frente; Honda (R$ 1,59 bi), Itaú (R$ 1,23 bi), BB (R$ 1,14 bi), Santander (R$ 0,99 bi).
- Ranking por receita de taxa de administração: BB (R$ 932 mi), Bradesco (R$ 845 mi), Honda (R$ 644 mi), Itaú (R$ 548 mi), Ademicon (R$ 524 mi) — ordem diferente da do PL, mostra que porte patrimonial e faturamento corrente não seguem o mesmo padrão.

### 2.3 Saúde financeira e casos de atenção

- **Nenhuma administradora do sistema opera com Patrimônio Líquido negativo** — indicador positivo de solidez geral do setor.
- **21 das 124 administradoras analisadas (17%) fecharam o período com resultado negativo** (receita operacional menor que despesa operacional). As margens mais negativas: KLUBI (−41,7%, também com a maior taxa de inadimplência de cotas do sistema, 31,6%), Mercabenco (−29,9%), FF Adm Cons (−1.147%, base de receita muito pequena — resultado distorcido por baixa escala).
- **Correlação entre inadimplência operacional (cotas) e margem financeira:** fraca e negativa (r ≈ −0,22). Ou seja, inadimplência de cotas explica só uma pequena parte do resultado financeiro — despesas administrativas e estrutura de custos pesam mais que a inadimplência isolada. Isso é uma informação relevante e pouco intuitiva.

### 2.4 Eficiência por cota administrada ("take rate" real)

Com o cruzamento carteira ativa (modelo) × receita de taxa de administração (BCB), foi possível calcular pela primeira vez uma receita média por cota sob gestão — hoje impossível de calcular só com dados operacionais. Entre administradoras com carteira relevante (>5.000 cotas), destacam-se Consórcio Volvo, Portobens e Scania com a maior receita por cota — perfil típico de administradoras de bens de maior ticket (veículos pesados/máquinas).

### 2.5 Limitações identificadas (importantes para a decisão da presidência)

- Duas administradoras de pequeno porte (M.I6, Brquality) apresentam PL superior ao Ativo Total apurado — inconsistência pontual de baixa materialidade, provavelmente por composição de contas não totalmente comparável entre instituições; não compromete os agregados do sistema, mas deve constar como nota técnica caso a tela seja publicada.
- Para administradoras que fazem parte de conglomerados financeiros maiores (ex. Scania, Volvo, captivas de bancos), o Patrimônio Líquido reportado ao BCB pode refletir capital da instituição financeira como um todo, não apenas da operação de consórcio — a métrica de "PL por cota" deve ser lida com essa ressalva, não como comparação direta de solidez.
- Esta é uma leitura de **uma única competência (03/2026)**; para conclusões de tendência (não só de fotografia) seria necessário acumular 2 a 3 competências — o portal do BCB permite isso.

---

## 3. Proposta de conteúdo para a nova página (5ª tela)

**Nome sugerido:** "Saúde Financeira das Administradoras" ou "Panorama Patrimonial (BCB)"

Estrutura sugerida, seguindo a linha visual já usada nas páginas Panorama/Documentação (cards institucionais, paleta do Comitê):

1. **Cabeçalho institucional** — mesmo padrão das demais páginas, com selo do Comitê e nota de fonte (BCB) e competência dos dados em destaque.
2. **Linha de KPIs do sistema** — PL total, Receita de Taxa de Administração total, Resultado agregado, % de administradoras com resultado negativo.
3. **Ranking de porte** — Top administradoras por PL e por receita de taxa de administração, lado a lado (nomes visíveis, conforme já definido).
4. **Indicador de concentração** — participação das 5 maiores vs. demais, com leitura textual objetiva (não HHI técnico exposto ao usuário final, mas a % de concentração sim).
5. **Cruzamento inédito: eficiência por cota** — receita de taxa de administração por cota sob gestão, por administradora, permitindo comparação de porte×faturamento pela primeira vez.
6. **Alerta de resultado negativo** — lista objetiva das administradoras com resultado negativo no período, sem juízo de valor, apenas o dado.
7. **Nota técnica/metodológica** — fonte (BCB, documento 4010), competência, e as limitações da seção 2.5 acima, resumidas em linguagem simples.
8. **Aviso legal reforçado** — além da cláusula já criada para as demais páginas, uma frase específica reforçando que os dados financeiros são públicos (BCB) e não configuram avaliação de crédito, rating ou recomendação.

---

## 4. Ponto central para a decisão da presidência

A diferença desta tela para o restante do material é qualitativa, não só de conteúdo: as páginas atuais mostram desempenho **operacional** (cotas, contemplação, inadimplência) — informação já pública e de leitura neutra. Esta nova tela exporia, por administradora nomeada, **resultado (lucro/prejuízo) e patrimônio líquido** — dado também público (BCB), mas de leitura mais sensível institucionalmente, especialmente a lista de administradoras com resultado negativo no período. Recomendo que a presidência avalie especificamente esse ponto antes da publicação, independentemente da qualidade técnica da análise.

---

## 5. Próximos passos sugeridos

Caso a presidência aprove o conceito, o próximo passo é a construção efetiva da página no arquivo Power BI (medidas DAX, nova tabela de fatos financeiros, visuais) — ainda não iniciada, aguardando sinalização para prosseguir.
