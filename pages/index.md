---
title: Relatório Executivo Institucional (CESOL)
description: Análise de performance financeira, unit economics e retenção estratégica.
---

# Resumo Executivo da Diretoria

Este relatório consolida os principais indicadores de rentabilidade, eficiência operacional e retenção de alunos. O foco é apoiar decisões de diretoria com base em evidências objetivas, priorizando previsibilidade de caixa, crescimento sustentável e redução de evasão.

<details>
  <summary style="font-weight: bold; cursor: pointer; margin-bottom: 1rem;">Navegação Rápida (Índice)</summary>
  <ul>
    <li><a href="#1-saúde-financeira-e-rentabilidade">1. Saúde Financeira e Rentabilidade</a></li>
    <li><a href="#2-prioridades-executivas-top-3-riscos-e-top-3-alavancas">2. Prioridades Executivas</a></li>
    <li><a href="#3-performance-por-segmento">3. Performance por Segmento</a></li>
    <li><a href="#4-retenção-e-gestão-de-churn">4. Retenção e Gestão de Churn</a></li>
    <li><a href="#5-unit-economics-e-eficiência-operacional">5. Unit Economics e Eficiência Operacional</a></li>
    <li><a href="#6-mix-comercial-e-qualidade-de-receita">6. Mix Comercial e Qualidade de Receita</a></li>
    <li><a href="#7-ponte-financeira-bruto-para-resultado">7. Ponte Financeira</a></li>
    <li><a href="#8-estrutura-de-custos">8. Estrutura de Custos</a></li>
    <li><a href="#9-impacto-financeiro-do-churn">9. Impacto Financeiro do Churn</a></li>
    <li><a href="#10-recomendação-ao-comitê-executivo">10. Recomendação Executiva</a></li>
  </ul>
</details>

```sql financial_health
with revenue as (
    select 
        sum(net_tuition) as total_revenue, 
        count(id) as active_students
    from cesol_data.alunos_ativos
),
costs as (
    select 
        sum(valor) as total_costs
    from cesol_data.custos_operacionais
)
select
    r.total_revenue,
    c.total_costs,
    (r.total_revenue - c.total_costs) as net_profit,
    (r.total_revenue - c.total_costs) / r.total_revenue as operational_margin,
    r.active_students
from revenue r
cross join costs c
```
## 1. Saúde Financeira e Rentabilidade

Manter margem operacional positiva e recorrente é condição básica para a sustentabilidade da operação. Os indicadores abaixo mostram o desempenho financeiro consolidado da base ativa de alunos.
<Grid cols=4>
<BigValue
data={financial_health}
value=total_revenue
fmt=brl
title="Faturamento (Receita Líquida)"
/>
<BigValue
data={financial_health}
value=total_costs
fmt=brl
title="Custos Operacionais"
/>
<BigValue
data={financial_health}
value=net_profit
fmt=brl
title="Resultado Operacional"
/>
<BigValue
data={financial_health}
value=operational_margin
fmt="pct"
title="Margem Operacional"
/>
</Grid>
<br>

> **<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="text-blue-600 inline-block align-text-bottom mr-1"><path d="M15 14c.2-1 .7-1.7 1.5-2.5 1-.9 1.5-2.2 1.5-3.5A6 6 0 0 0 6 8c0 1 .2 2.2 1.5 3.5.7.9 1.2 1.5 1.5 2.5"/><path d="M9 18h6"/><path d="M10 22h4"/></svg> Insight Estratégico**
> Atualmente, com uma base de <Value data={financial_health} column=active_students /> alunos ativos, a instituição opera com margem de **<Value data={financial_health} column=operational_margin fmt=pct />** e faturamento líquido mensal de **<Value data={financial_health} column=total_revenue fmt=brl />**.

```sql top_risks
with churn_risk as (
    select
        'Churn: ' || Motivo as item,
        Quantidade * (select avg(coalesce(net_tuition, 0)) from cesol_data.alunos_ativos) as impacto_financeiro_mensal,
        'Receita em risco por evasão' as tipo
    from cesol_data.churn_motivos
),
discount_risk as (
    select
        'Descontos: ' || segment as item,
        sum(coalesce(discount_value, 0) + coalesce(scholarship_value, 0)) as impacto_financeiro_mensal,
        'Compressão de margem por descontos e bolsas' as tipo
    from cesol_data.alunos_ativos
    group by segment
),
cost_risk as (
    select
        'Custos: ' || coalesce(categoria, 'Sem Categoria') as item,
        sum(coalesce(valor, 0)) as impacto_financeiro_mensal,
        'Pressão de custo operacional' as tipo
    from cesol_data.custos_operacionais
    group by coalesce(categoria, 'Sem Categoria')
),
all_risks as (
    select * from churn_risk
    union all
    select * from discount_risk
    union all
    select * from cost_risk
)
select
    item,
    tipo,
    impacto_financeiro_mensal
from all_risks
order by impacto_financeiro_mensal desc
limit 3
```

```sql top_levers
with churn_lever as (
    select
        'Recuperação de churn: ' || Motivo as item,
        Quantidade * (select avg(coalesce(net_tuition, 0)) from cesol_data.alunos_ativos) as valor_potencial_mensal,
        'Mitigar evasão nos principais motivos' as tipo
    from cesol_data.churn_motivos
),
pricing_lever as (
    select
        'Otimização de preço: ' || segment as item,
        avg(coalesce(net_tuition, 0)) * count(*) * 0.05 as valor_potencial_mensal,
        'Ganho potencial com melhoria de ticket em 5%' as tipo
    from cesol_data.alunos_ativos
    group by segment
),
discount_lever as (
    select
        'Eficiência comercial: ' || segment as item,
        sum(coalesce(discount_value, 0) + coalesce(scholarship_value, 0)) * 0.1 as valor_potencial_mensal,
        'Redução potencial de 10% em descontos e bolsas' as tipo
    from cesol_data.alunos_ativos
    group by segment
),
all_levers as (
    select * from churn_lever
    union all
    select * from pricing_lever
    union all
    select * from discount_lever
)
select
    item,
    tipo,
    valor_potencial_mensal
from all_levers
order by valor_potencial_mensal desc
limit 3
```

## 2. Prioridades Executivas (Top 3 Riscos e Top 3 Alavancas)

As prioridades abaixo são geradas automaticamente a partir dos dados operacionais e financeiros, com foco em impacto mensal estimado.

<Grid cols=2>
<BarChart
data={top_risks}
x=item
y=impacto_financeiro_mensal
series=item
title="Top 3 Riscos Financeiros"
yFmt=brl
swapXY=true
labels=true
/>
<BarChart
data={top_levers}
x=item
y=valor_potencial_mensal
series=item
title="Top 3 Alavancas de Valor"
yFmt=brl
swapXY=true
labels=true
/>
</Grid>

<Grid cols=2>
<details><summary style="cursor: pointer; opacity: 0.8;">Ver tabela de riscos</summary><DataTable data={top_risks} /></details>
<details><summary style="cursor: pointer; opacity: 0.8;">Ver tabela de alavancas</summary><DataTable data={top_levers} /></details>
</Grid>

## 3. Performance por Segmento

A análise por segmento orienta estratégia comercial, alocação de marketing e política de precificação. O objetivo é equilibrar volume de alunos, ticket médio e faturamento total.
```sql segment_performance
select
    segment as segmento,
    sum(net_tuition) as faturamento_total,
    avg(net_tuition) as ticket_medio,
    count(id) as volume_alunos
from cesol_data.alunos_ativos
group by segment
order by faturamento_total desc
```
<BarChart
data={segment_performance}
x=segmento
y=faturamento_total
series=segmento
title="Faturamento por Etapa de Ensino"
yFmt=brl
swapXY=true
labels=true
/>

> **<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="text-blue-600 inline-block align-text-bottom mr-1"><circle cx="12" cy="12" r="10"/><circle cx="12" cy="12" r="6"/><circle cx="12" cy="12" r="2"/></svg> Oportunidade de Ticket**
> O segmento **<Value data={segment_performance} row=0 column=segmento />** lidera em faturamento, com <Value data={segment_performance} row=0 column=faturamento_total fmt=brl />. Em precificação, o maior ticket médio está em **<Value data={segment_performance} column=segmento order="ticket_medio desc" row=0 />**, com <Value data={segment_performance} column=ticket_medio order="ticket_medio desc" row=0 fmt=brl /> por aluno.

## 4. Retenção e Gestão de Churn

Retenção é vetor crítico de resultado, pois o custo de aquisição tende a ser superior ao custo de permanência. A leitura abaixo prioriza os principais motivos de cancelamento para orientar planos de mitigação.
```sql churn_analysis
select
    Motivo as motivo,
    Quantidade as quantidade,
    Quantidade * 1.0 / (select sum(Quantidade) from cesol_data.churn_motivos) as peso_percentual
from cesol_data.churn_motivos
order by Quantidade desc
```
<ECharts config={
{
tooltip: { trigger: 'item', formatter: '{b}: {c} casos' },
legend: { show: false },
series:[
{
name: 'Motivos de Evasão',
type: 'pie',
center: ['50%', '50%'],
radius:['40%', '68%'],
avoidLabelOverlap: true,
label: { show: true, formatter: '{b}', position: 'outside' },
labelLine: { show: true, length: 10, length2: 12 },
itemStyle: { borderRadius: 8, borderColor: '#fff', borderWidth: 2 },
data: churn_analysis.map(d => ({ value: d.quantidade, name: d.motivo }))
}
]
}
} />

> **<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="text-yellow-600 inline-block align-text-bottom mr-1"><path d="m21.73 18-8-14a2 2 0 0 0-3.48 0l-8 14A2 2 0 0 0 4 21h16a2 2 0 0 0 1.73-3Z"/><path d="M12 9v4"/><path d="M12 17h.01"/></svg> Alerta de Evasão**
> O principal motivo de evasão é **<Value data={churn_analysis} row=0 column=motivo />**, que representa **<Value data={churn_analysis} row=0 column=peso_percentual fmt=pct />** das saídas. A recomendação é estruturar plano de ação específico para esse fator, com *ownership* entre Diretoria Acadêmica e time de Retenção, metas quinzenais e monitoramento contínuo de impacto.

```sql unit_economics
with base as (
    select
        count(*) as alunos_ativos,
        avg(coalesce(net_tuition, 0)) as ticket_medio_liquido,
        avg(coalesce(full_tuition, 0)) as ticket_medio_bruto,
        avg(coalesce(discount_value, 0) + coalesce(scholarship_value, 0)) as desconto_medio
    from cesol_data.alunos_ativos
),
custos as (
    select
        sum(coalesce(valor, 0)) as custo_total
    from cesol_data.custos_operacionais
)
select
    b.alunos_ativos,
    b.ticket_medio_liquido,
    b.ticket_medio_bruto,
    b.desconto_medio,
    c.custo_total,
    c.custo_total / nullif(b.alunos_ativos, 0) as custo_por_aluno,
    b.ticket_medio_liquido - (c.custo_total / nullif(b.alunos_ativos, 0)) as margem_contribuicao_por_aluno
from base b
cross join custos c
```

## 5. Unit Economics e Eficiência Operacional

Esta seção detalha eficiência unitária da operação para suportar decisões de crescimento com disciplina financeira.

<Grid cols=4>
<BigValue
data={unit_economics}
value=ticket_medio_liquido
fmt=brl
title="Ticket Médio Líquido"
/>
<BigValue
data={unit_economics}
value=custo_por_aluno
fmt=brl
title="Custo Operacional por Aluno"
/>
<BigValue
data={unit_economics}
value=margem_contribuicao_por_aluno
fmt=brl
title="Margem de Contribuição por Aluno"
/>
<BigValue
data={unit_economics}
value=desconto_medio
fmt=brl
title="Desconto Médio por Aluno"
/>
</Grid>

```sql segment_mix
select
    segment as segmento,
    count(*) as alunos,
    sum(coalesce(net_tuition, 0)) as receita_liquida,
    avg(coalesce(net_tuition, 0)) as ticket_medio_liquido,
    sum(coalesce(discount_value, 0) + coalesce(scholarship_value, 0)) as desconto_total
from cesol_data.alunos_ativos
group by segment
order by receita_liquida desc
```

## 6. Mix Comercial e Qualidade de Receita

<Grid cols=2>
<BarChart
data={segment_mix}
x=segmento
y=receita_liquida
series=segmento
title="Receita Líquida por Segmento"
yFmt=brl
swapXY=true
labels=true
/>
<BarChart
data={segment_mix}
x=segmento
y=ticket_medio_liquido
series=segmento
title="Ticket Médio Líquido"
yFmt=brl
swapXY=true
labels=true
/>
</Grid>

<details><summary style="cursor: pointer; opacity: 0.8; margin-top: 1rem;">Ver tabela detalhada do Mix Comercial</summary><DataTable data={segment_mix} search=true /></details>

> **<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="text-blue-600 inline-block align-text-bottom mr-1"><circle cx="12" cy="12" r="10"/><path d="M12 16v-4"/><path d="M12 8h.01"/></svg> Nota de Qualidade:** O cruzamento de receita, ticket e descontos por segmento permite separar crescimento com valor de crescimento com pressão de preço.

```sql financial_bridge
with receita as (
    select
        sum(coalesce(full_tuition, 0)) as receita_bruta,
        sum(coalesce(discount_value, 0) + coalesce(scholarship_value, 0)) as descontos_bolsas,
        sum(coalesce(net_tuition, 0)) as receita_liquida
    from cesol_data.alunos_ativos
),
custos as (
    select
        sum(coalesce(valor, 0)) as custos
    from cesol_data.custos_operacionais
)
select 'Receita Bruta' as etapa, receita_bruta as valor from receita
union all
select 'Descontos e Bolsas' as etapa, -descontos_bolsas as valor from receita
union all
select 'Receita Líquida' as etapa, receita_liquida as valor from receita
union all
select 'Custos Operacionais' as etapa, -custos as valor from custos
union all
select 'Resultado Operacional' as etapa, (select receita_liquida from receita) - (select custos from custos) as valor
```

## 7. Ponte Financeira (Bruto para Resultado)

<BarChart
data={financial_bridge}
x=etapa
y=valor
title="Ponte de Formação do Resultado"
yFmt=brl
labels=true
fillColor={["#2E7D32", "#C0392B", "#2E7D32", "#C0392B", "#000000"]}
/>

```sql cost_structure
select
    coalesce(categoria, 'Sem Categoria') as categoria,
    sum(coalesce(valor, 0)) as custo_total
from cesol_data.custos_operacionais
group by 1
order by custo_total desc
```

## 8. Estrutura de Custos

<ECharts config={{
tooltip: { trigger: 'item', formatter: '{b}: {c}' },
legend: { show: false },
series: [
{
name: 'Custos',
type: 'pie',
center: ['50%', '50%'],
radius: ['40%', '68%'],
avoidLabelOverlap: true,
label: { show: true, formatter: '{b}', position: 'outside' },
labelLine: { show: true, length: 10, length2: 12 },
itemStyle: { borderRadius: 8, borderColor: '#fff', borderWidth: 2 },
data: cost_structure.map(d => ({ value: d.custo_total, name: d.categoria }))
}
]
}} />

```sql churn_financial_impact
with ticket as (
    select avg(coalesce(net_tuition, 0)) as ticket_medio
    from cesol_data.alunos_ativos
),
churn as (
    select
        Motivo as motivo,
        Quantidade as quantidade
    from cesol_data.churn_motivos
)
select
    c.motivo,
    c.quantidade,
    c.quantidade * t.ticket_medio as perda_receita_mensal_estimada
from churn c
cross join ticket t
order by perda_receita_mensal_estimada desc
```

## 9. Impacto Financeiro do Churn

<BarChart
data={churn_financial_impact}
x=motivo
y=perda_receita_mensal_estimada
series=motivo
title="Perda de Receita Mensal Estimada por Motivo de Churn"
yFmt=brl
swapXY=true
labels=true
/>

<details><summary style="cursor: pointer; opacity: 0.8;">Ver tabela de impacto do churn</summary><DataTable data={churn_financial_impact} search=true /></details>

> **<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="text-gray-600 inline-block align-text-bottom mr-1"><line x1="18" x2="18" y1="20" y2="10"/><line x1="12" x2="12" y1="20" y2="4"/><line x1="6" x2="6" y1="20" y2="14"/></svg> Avaliação Financeira:** A priorização de retenção deve considerar não apenas volume de casos, mas também o impacto financeiro potencial por motivo.

## 10. Recomendação ao Comitê Executivo

```sql plano_3090
with base as (
    select
        (select sum(coalesce(net_tuition, 0)) from cesol_data.alunos_ativos) as receita_liquida_atual,
        (select sum(coalesce(valor, 0)) from cesol_data.custos_operacionais) as custos_atuais,
        (select sum(coalesce(discount_value, 0) + coalesce(scholarship_value, 0)) from cesol_data.alunos_ativos) as descontos_atuais,
        (select sum(Quantidade) from cesol_data.churn_motivos) as churn_casos_atuais
),
metas as (
    select
        '30 dias' as fase,
        'Retenção (churn)' as alavanca,
        'Casos de churn' as kpi,
        churn_casos_atuais as baseline,
        churn_casos_atuais * 0.92 as meta,
        'Redução de 8% em casos de churn' as objetivo
    from base
    union all
    select
        '30 dias' as fase,
        'Eficiência comercial' as alavanca,
        'Descontos e bolsas (R$)' as kpi,
        descontos_atuais as baseline,
        descontos_atuais * 0.97 as meta,
        'Redução de 3% em descontos e bolsas' as objetivo
    from base
    union all
    select
        '30 dias' as fase,
        'Margem operacional' as alavanca,
        'Margem operacional (%)' as kpi,
        ((receita_liquida_atual - custos_atuais) / nullif(receita_liquida_atual, 0)) * 100 as baseline,
        (((receita_liquida_atual * 1.01) - (custos_atuais * 0.99)) / nullif((receita_liquida_atual * 1.01), 0)) * 100 as meta,
        'Ganho mínimo de 1 p.p. na margem operacional' as objetivo
    from base
    union all
    select
        '60 dias' as fase,
        'Retenção (churn)' as alavanca,
        'Casos de churn' as kpi,
        churn_casos_atuais as baseline,
        churn_casos_atuais * 0.85 as meta,
        'Redução de 15% em casos de churn' as objetivo
    from base
    union all
    select
        '60 dias' as fase,
        'Eficiência comercial' as alavanca,
        'Descontos e bolsas (R$)' as kpi,
        descontos_atuais as baseline,
        descontos_atuais * 0.93 as meta,
        'Redução de 7% em descontos e bolsas' as objetivo
    from base
    union all
    select
        '60 dias' as fase,
        'Margem operacional' as alavanca,
        'Margem operacional (%)' as kpi,
        ((receita_liquida_atual - custos_atuais) / nullif(receita_liquida_atual, 0)) * 100 as baseline,
        (((receita_liquida_atual * 1.03) - (custos_atuais * 0.97)) / nullif((receita_liquida_atual * 1.03), 0)) * 100 as meta,
        'Ganho acumulado de 2 p.p. na margem operacional' as objetivo
    from base
    union all
    select
        '90 dias' as fase,
        'Retenção (churn)' as alavanca,
        'Casos de churn' as kpi,
        churn_casos_atuais as baseline,
        churn_casos_atuais * 0.80 as meta,
        'Redução de 20% em casos de churn' as objetivo
    from base
    union all
    select
        '90 dias' as fase,
        'Eficiência comercial' as alavanca,
        'Descontos e bolsas (R$)' as kpi,
        descontos_atuais as baseline,
        descontos_atuais * 0.90 as meta,
        'Redução de 10% em descontos e bolsas' as objetivo
    from base
    union all
    select
        '90 dias' as fase,
        'Margem operacional' as alavanca,
        'Margem operacional (%)' as kpi,
        ((receita_liquida_atual - custos_atuais) / nullif(receita_liquida_atual, 0)) * 100 as baseline,
        (((receita_liquida_atual * 1.05) - (custos_atuais * 0.95)) / nullif((receita_liquida_atual * 1.05), 0)) * 100 as meta,
        'Ganho acumulado de 3 p.p. na margem operacional' as objetivo
    from base
)
select * from metas
```

A recomendação para o comitê executivo é estruturar a execução em um ciclo 30-60-90 dias, com metas quantitativas por alavanca e governança quinzenal de performance.

```sql plano_3090_cards
with base as (
    select
        (select sum(coalesce(net_tuition, 0)) from cesol_data.alunos_ativos) as receita_liquida_atual,
        (select sum(coalesce(valor, 0)) from cesol_data.custos_operacionais) as custos_atuais,
        (select sum(coalesce(discount_value, 0) + coalesce(scholarship_value, 0)) from cesol_data.alunos_ativos) as descontos_atuais,
        (select sum(Quantidade) from cesol_data.churn_motivos) as churn_casos_atuais
),
metas as (
    select
        '30 dias' as fase,
        'AMARELO' as semaforo,
        0.08 as churn_reducao_pct,
        0.03 as desconto_reducao_pct,
        1.0 as ganho_margem_pp
    from base
    union all
    select
        '60 dias' as fase,
        'AMARELO' as semaforo,
        0.15 as churn_reducao_pct,
        0.07 as desconto_reducao_pct,
        2.0 as ganho_margem_pp
    from base
    union all
    select
        '90 dias' as fase,
        'VERDE' as semaforo,
        0.20 as churn_reducao_pct,
        0.10 as desconto_reducao_pct,
        3.0 as ganho_margem_pp
    from base
)
select *
from metas
order by case fase when '30 dias' then 1 when '60 dias' then 2 else 3 end
```

<Grid cols=3>
<div class="bg-gray-50 dark:bg-gray-800 rounded-lg p-6 shadow-sm border border-gray-200 dark:border-gray-700">
  <h3 style="margin-top:0;" class="text-xl font-bold mb-4">Fase 30 dias</h3>
  <div class="mb-4">Status de meta: <span class="inline-block px-3 py-1 rounded-full text-xs font-bold bg-yellow-600 text-white">AMARELO</span></div>
  <ul class="space-y-2 text-sm">
    <li><span class="font-semibold">Redução de churn:</span> <Value data={plano_3090_cards} row=0 column=churn_reducao_pct fmt=pct /></li>
    <li><span class="font-semibold">Redução de descontos:</span> <Value data={plano_3090_cards} row=0 column=desconto_reducao_pct fmt=pct /></li>
    <li><span class="font-semibold">Ganho de margem:</span> <Value data={plano_3090_cards} row=0 column=ganho_margem_pp /> p.p.</li>
  </ul>
</div>
<div class="bg-gray-50 dark:bg-gray-800 rounded-lg p-6 shadow-sm border border-gray-200 dark:border-gray-700">
  <h3 style="margin-top:0;" class="text-xl font-bold mb-4">Fase 60 dias</h3>
  <div class="mb-4">Status de meta: <span class="inline-block px-3 py-1 rounded-full text-xs font-bold bg-yellow-600 text-white">AMARELO</span></div>
  <ul class="space-y-2 text-sm">
    <li><span class="font-semibold">Redução de churn:</span> <Value data={plano_3090_cards} row=1 column=churn_reducao_pct fmt=pct /></li>
    <li><span class="font-semibold">Redução de descontos:</span> <Value data={plano_3090_cards} row=1 column=desconto_reducao_pct fmt=pct /></li>
    <li><span class="font-semibold">Ganho de margem:</span> <Value data={plano_3090_cards} row=1 column=ganho_margem_pp /> p.p.</li>
  </ul>
</div>
<div class="bg-gray-50 dark:bg-gray-800 rounded-lg p-6 shadow-sm border border-gray-200 dark:border-gray-700">
  <h3 style="margin-top:0;" class="text-xl font-bold mb-4">Fase 90 dias</h3>
  <div class="mb-4">Status de meta: <span class="inline-block px-3 py-1 rounded-full text-xs font-bold bg-green-700 text-white">VERDE</span></div>
  <ul class="space-y-2 text-sm">
    <li><span class="font-semibold">Redução de churn:</span> <Value data={plano_3090_cards} row=2 column=churn_reducao_pct fmt=pct /></li>
    <li><span class="font-semibold">Redução de descontos:</span> <Value data={plano_3090_cards} row=2 column=desconto_reducao_pct fmt=pct /></li>
    <li><span class="font-semibold">Ganho de margem:</span> <Value data={plano_3090_cards} row=2 column=ganho_margem_pp /> p.p.</li>
  </ul>
</div>
</Grid>

<details><summary style="cursor: pointer; opacity: 0.8; margin-top: 1rem;">Ver tabela detalhada de metas 30-60-90</summary><DataTable data={plano_3090} /></details>

```sql plano_3090_semaforo
select '30 dias' as fase, 'Churn' as kpi, 2 as status_valor, 'AMARELO' as status_texto
union all
select '30 dias' as fase, 'Descontos' as kpi, 2 as status_valor, 'AMARELO' as status_texto
union all
select '30 dias' as fase, 'Margem' as kpi, 1 as status_valor, 'VERMELHO' as status_texto
union all
select '60 dias' as fase, 'Churn' as kpi, 2 as status_valor, 'AMARELO' as status_texto
union all
select '60 dias' as fase, 'Descontos' as kpi, 2 as status_valor, 'AMARELO' as status_texto
union all
select '60 dias' as fase, 'Margem' as kpi, 2 as status_valor, 'AMARELO' as status_texto
union all
select '90 dias' as fase, 'Churn' as kpi, 3 as status_valor, 'VERDE' as status_texto
union all
select '90 dias' as fase, 'Descontos' as kpi, 3 as status_valor, 'VERDE' as status_texto
union all
select '90 dias' as fase, 'Margem' as kpi, 3 as status_valor, 'VERDE' as status_texto
```

<ECharts config={{
grid: { top: 20, right: 20, bottom: 60, left: 80 },
tooltip: { trigger: 'item', formatter: (p) => `${['30 dias', '60 dias', '90 dias'][p.data[0]]} | ${['Churn', 'Descontos', 'Margem'][p.data[1]]}: ${p.data[3]}` },
xAxis: { type: 'category', data: ['30 dias', '60 dias', '90 dias'] },
yAxis: { type: 'category', data: ['Churn', 'Descontos', 'Margem'] },
visualMap: {
  min: 1,
  max: 3,
  calculable: false,
  orient: 'horizontal',
  left: 'center',
  bottom: 0,
  inRange: { color: ['#C0392B', '#B7791F', '#2E7D32'] },
  text: ['Verde', 'Vermelho']
},
series: [{
  type: 'heatmap',
  data: plano_3090_semaforo.map(d => {
    const xIndex = ['30 dias', '60 dias', '90 dias'].indexOf(d.fase);
    const yIndex = ['Churn', 'Descontos', 'Margem'].indexOf(d.kpi);
    return [xIndex, yIndex, d.status_valor, d.status_texto];
  }),
  label: {
    show: true,
    formatter: (p) => p.data[3],
    color: '#000000',
    fontWeight: 'bold',
    textBorderColor: '#ffffff',
    textBorderWidth: 2
  },
  itemStyle: { borderColor: '#888888', borderWidth: 1 }
}]
}} />

```sql plano_3090_resumo
with base as (
    select
        (select sum(coalesce(discount_value, 0) + coalesce(scholarship_value, 0)) from cesol_data.alunos_ativos) as descontos_atuais,
        (select sum(Quantidade) from cesol_data.churn_motivos) as churn_casos_atuais
),
metas as (
    select '30 dias' as fase, churn_casos_atuais as churn_baseline, churn_casos_atuais * 0.92 as churn_meta, descontos_atuais as descontos_baseline, descontos_atuais * 0.97 as descontos_meta from base
    union all
    select '60 dias' as fase, churn_casos_atuais as churn_baseline, churn_casos_atuais * 0.85 as churn_meta, descontos_atuais as descontos_baseline, descontos_atuais * 0.93 as descontos_meta from base
    union all
    select '90 dias' as fase, churn_casos_atuais as churn_baseline, churn_casos_atuais * 0.80 as churn_meta, descontos_atuais as descontos_baseline, descontos_atuais * 0.90 as descontos_meta from base
)
select
    fase,
    churn_baseline,
    churn_meta,
    descontos_baseline,
    descontos_meta
from metas
order by case fase when '30 dias' then 1 when '60 dias' then 2 else 3 end
```

<LineChart
data={plano_3090_resumo}
x=fase
y=churn_baseline
y2=churn_meta
title="Trajetória de Redução de Churn: Baseline vs Meta"
/>

<LineChart
data={plano_3090_resumo}
x=fase
y=descontos_baseline
y2=descontos_meta
title="Trajetória de Redução de Descontos: Baseline vs Meta"
yFmt=brl
/>

> **<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="text-green-600 inline-block align-text-bottom mr-1"><path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"/><path d="m9 11 3 3L22 4"/></svg> Próximos Passos (Deliberação)**
> Recomenda-se aprovar imediatamente os *owners* por alavanca, o quadro de metas acima e um rito quinzenal com avaliação de desvio, correção de rota e reporte consolidado à presidência.
