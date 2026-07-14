# SPEC2 — Especificações do index.html

## Visão Geral

Painel de signage/dashboard para monitoramento de manutenção da **SEINFRA/UFG** (Superintendência de Infraestrutura — Universidade Federal de Goiás). Apresentação em tela cheia com rotação automática de 5 slides. Os dados são carregados dinamicamente de `data.json` via fetch a cada 30 minutos.

## Fonte de Dados

- Arquivo: `data.json` (gerado localmente a partir de `manutencao_07_2026.json`)
- Carregamento: `fetch('data.json?t=' + Date.now())` com cache-busting
- Atualização automática: `setInterval(fetchDashboardData, 30 * 60 * 1000)`
- Tela de loading: exibida durante o carregamento (`#sg-loading`)

## Estrutura do Layout

```
┌─────────────────────────────────────────┐
│         LOADING OVERLAY (z:9999)        │
├─────────────────────────────────────────┤
│              HEADER (72px)              │
├─────────────────────────────────────────┤
│                                         │
│           SLIDES CONTAINER              │
│          (5 slides, overflow)           │
│                                         │
├─────────────────────────────────────────┤
│              FOOTER (52px)              │
└─────────────────────────────────────────┘
```

- **Loading Overlay**: Tela fixa com "SEINFRA / UFG" + "Sincronizando dados com o servidor..." enquanto busca `data.json`
- **Header**: Logo SEINFRA/UFG (sem subtítulo), título do slide (centralizado), status "DADOS ATIVOS" (com pulso animado), relógio em tempo real
- **Footer**: Fonte dos dados, 5 dots de navegação, controles Pause/Tela Cheia

## Slides (5 total)

### Slide 1 — Visão Geral — Histórico 2019-2026

| Componente | Dados |
|------------|-------|
| KPI Row (4 colunas) | Total Requisições, Total Horas, Finalizadas (%), Em Aberto (%) — valores do período completo |
| Tabela "Top Oficinas por Requisições" | 8 oficinas ranqueadas por req., colunas: #, Oficina, Req., % |
| Tabela "Top Unidades Requisitantes" | 8 unidades ranqueadas por req., colunas: #, Unidade, Req. |

- KPIs preenchidos via `D.kpis` (`total_requisicoes`, `total_horas`, `total_finalizadas`, `total_abertas`)
- Tabelas via `D.ofic_req` e `D.unidades`

### Slide 2 — Visão Geral — Ano Corrente (2026)

| Componente | Dados |
|------------|-------|
| KPI Row (4 colunas) | Req., Horas, Finalizadas (%), Em Aberto (%) — filtrados para 2026 |
| Tabela "Top Oficinas por Requisições (2026)" | 8 oficinas, colunas: #, Oficina, Req., % |
| Tabela "Top Unidades Requisitantes (2026)" | 8 unidades, colunas: #, Unidade, Req. |

- KPIs preenchidos via `D.kpis_2026` (`req`, `horas`, `fin`, `abertas`)
- Percentuais calculados dinamicamente: `(fin / req) * 100`
- Tabelas via `D.ofic_req_2026` e `D.unidades_2026`

### Slide 3 — Visão Geral — Ano Anterior (2025)

| Componente | Dados |
|------------|-------|
| KPI Row (4 colunas) | Req., Horas, Finalizadas (%), Em Aberto (%) — filtrados para 2025 |
| Tabela "Top Oficinas por Requisições (2025)" | 8 oficinas, colunas: #, Oficina, Req., % |
| Tabela "Top Unidades Requisitantes (2025)" | 8 unidades, colunas: #, Unidade, Req. |

- KPIs preenchidos via `D.kpis_2025` (`req`, `horas`, `fin`, `abertas`)
- Tabelas via `D.ofic_req_2025` e `D.unidades_2025`

### Slide 4 — Evolução Anual de Requisições

| Componente | Tipo | Dados |
|------------|------|-------|
| Evolução Anual | Bar (full-width) | Requisições por ano (2019–2026), ano corrente em cyan, demais em azul |

- Chart: `D.yearly_req`
- Plugin `datalabelsPlugin` exibe valores acima de cada barra
- Título com estilo inline: 22px, bold, branco; subtítulo 16px

### Slide 5 — Divisão: Equipamentos vs Predial

| Componente | Tipo | Dados |
|------------|------|-------|
| Divisão Anual | Bar agrupado (full-width) | Dois datasets: Predial (`#FFB300`) e Equipamentos (`#00E676`) |

- Charts: `D.yearly_divisao[ano].Predial`, `D.yearly_divisao[ano].Equipamentos`
- Plugin `datalabelsPlugin` exibe valores acima de cada barra
- Título com estilo inline: 22px, bold, branco; subtítulo 16px

## Estrutura de Dados (`data.json` → objeto `D`)

```javascript
// KPIs globais (histórico)
D.kpis.total_requisicoes   // number
D.kpis.total_horas         // number
D.kpis.total_finalizadas   // number
D.kpis.total_abertas       // number
D.kpis.mediana_sla         // number (dias) — definido mas não utilizado no HTML
D.kpis.media_horas_req     // number — definido mas não utilizado no HTML

// KPIs por ano
D.kpis_2026.req            // number
D.kpis_2026.horas          // number
D.kpis_2026.fin            // number
D.kpis_2026.abertas        // number
D.kpis_2025.req            // number
D.kpis_2025.horas          // number
D.kpis_2025.fin            // number
D.kpis_2025.abertas        // number

// Tabelas de oficinas (req.)
D.ofic_req                 // { "Elétrica": N, ... } — histórico
D.ofic_req_2026            // { ... } — 2026
D.ofic_req_2025            // { ... } — 2025

// Tabelas de unidades (req.)
D.unidades                 // { "Escola De Agronomia": N, ... } — histórico
D.unidades_2026            // { ... } — 2026
D.unidades_2025            // { ... } — 2025

// Oficinas por horas (definido mas não renderizado)
D.ofic_horas               // { "Elétrica": 101754, ... }

// Status (definido mas não renderizado)
D.status                   // { "Finalizada": 109767, ... }

// Evolução anual
D.yearly_req               // { "2019": 5294, ..., "2026": 5428 }
D.yearly_divisao           // { "2019": { "Predial": N, "Equipamentos": N }, ... }
```

## Comportamentos

| Funcionalidade | Detalhe |
|----------------|---------|
| Carregamento inicial | `fetchDashboardData()` chamado no init + `goToSlide(0)` |
| Auto-refresh | A cada 30 minutos, recarrega `data.json` e re-renderiza tudo |
| Tela de loading | Overlay `#sg-loading` visível durante fetch; ocultado ao completar |
| Erro de fetch | Mensagem de erro vermelha exibida no overlay com detalhes HTTP |
| Rotação automática | **15 segundos** por slide (`SLIDE_DURATION = 15000`) |
| Transição | Fade in/out com `opacity 0.6s ease` |
| Animação de entrada | `fadeUp` 0.5s com delays escalonados (0.05s–0.35s) |
| Relógio | Atualização a cada 1s, formato HH:MM:SS + data por extenso |
| Pause/Play | Alterna rotação automática, botão no footer |
| Tela Cheia | `requestFullscreen()` / `exitFullscreen()` |
| Navegação por dots | Clique nos dots do footer navega diretamente ao slide |
| Re-render | `renderDashboard()` limpa DOM via helper `safe()` e reconstrói tabelas e charts |
| Destruição de charts | Cada chart chama `.destroy()` antes de recriar |

## Funções Principais

| Função | Responsabilidade |
|--------|------------------|
| `fetchDashboardData()` | Busca `data.json`, popula `D`, chama `renderDashboard()` e atualiza KPIs |
| `renderDashboard()` | Limpa DOM (via `safe()`) e chama funções de build ativas |
| `buildOficinas()` | Tabela Slide 1 — oficinas históricas |
| `buildUnidades()` | Tabela Slide 1 — unidades históricas |
| `buildOficinas2026()` | Tabela Slide 2 — oficinas 2026 |
| `buildUnidades2026()` | Tabela Slide 2 — unidades 2026 |
| `buildOficinas2025()` | Tabela Slide 3 — oficinas 2025 |
| `buildUnidades2025()` | Tabela Slide 3 — unidades 2025 |
| `buildChartEvolucaoAnual()` | Bar Slide 4 — evolução anual com datalabels |
| `buildChartDivisaoAnual()` | Bar agrupado Slide 5 — Predial vs Equipamentos com datalabels |
| `buildChartsSlide6()` | Wrapper (legacy) — chama `buildChartEvolucaoAnual()` + `buildChartDivisaoAnual()` |
| `goToSlide(idx)` | Navega para slide específico, atualiza dots e título |
| `togglePause()` | Pausa/retoma rotação automática |
| `toggleFullscreen()` | Alterna modo tela cheia |

### Funções definidas mas NÃO utilizadas no HTML atual

| Função | Observação |
|--------|-----------|
| `buildChartStatus()` | Canvas `#chartStatus` não existe no HTML |
| `buildChartOficinas()` | Canvas `#chartOficinas` não existe no HTML |
| `buildChartAnual()` | Canvas `#chartAnual` não existe no HTML |
| `buildBarOficinas()` | Container `#barOficinaHoras` não existe no HTML |

## Plugin Customizado

### `datalabelsPlugin`

Plugin inline que exibe valores numéricos acima de cada barra nos gráficos Slides 4 e 5. Formata valores ≥1000 como `Xk` (ex: `18,7k`). Fonte: bold 14px Inter, cor branca.

## Dependências Externas

| Recurso | Versão |
|---------|--------|
| Google Fonts — Inter | 300–900 |
| Google Fonts — JetBrains Mono | 400–600 |
| Chart.js | 4.4.1 (UMD via CDN) |

## Paleta de Cores

| Token | Hex | Uso |
|-------|-----|-----|
| `--bg` | `#09090D` | Fundo principal |
| `--surface` | `#111118` | Cards, header, footer |
| `--surface2` | `#1A1A24` | Superfícies secundárias, tracks |
| `--surface3` | `#22222F` | Hover states |
| `--border` | `#252535` | Bordas principais |
| `--border2` | `#333348` | Bordas secundárias |
| `--text` | `#F0F0FA` | Texto principal |
| `--text2` | `#9090B0` | Texto secundário |
| `--text3` | `#5A5A7A` | Texto terciário, labels |
| `--cyan` | `#00D4FF` | Destaque principal, KPIs, ano corrente |
| `--ok` | `#00E676` | Status positivo, finalizadas, Equipamentos |
| `--warn` | `#FFB300` | Atenção, Predial |
| `--danger` | `#FF4444` | Crítico, em aberto |
| `--blue` | `#4B8BFF` | Secundário, barras padrão |
| `--purple` | `#9B59FF` | Terciário |

## Tipografia

- **Fonte principal**: Inter (sans-serif)
- **Fonte monoespaçada**: JetBrains Mono (valores numéricos, relógio)
- **Logo**: 22px, weight 800, cyan com text-shadow
- **Título do slide (header center)**: 16px, weight 800
- **KPI label**: 13px, weight 800, uppercase
- **KPI value**: 32px, weight 800, JetBrains Mono
- **KPI sub**: 12px, weight 500
- **KPI Big label**: 14px, weight 700, uppercase (mantido no CSS, sem uso no HTML)
- **KPI Big value**: 68px, weight 800, JetBrains Mono (mantido no CSS, sem uso no HTML)
- **Card title**: 18px, weight 800 (CSS); slides 4–5 usam inline 22px bold
- **Card subtitle**: 14px, weight 500 (CSS); slides 4–5 usam inline 16px
- **Table header (th)**: 13px, weight 800, uppercase
- **Table cell (td)**: 15px
- **Table mono cell**: 14px
- **Pill/badge**: 13px, weight 600
- **Chart axis labels**: 14–15px (slides 4–5 maiores que os defaults)

## Diferenças em Relação ao index2.html

| Aspecto | index.html | index2.html |
|---------|-----------|-------------|
| Slides | **5** | 6 |
| Duração por slide | **15s** | 25s |
| Slide SLA | **Removido** | Presente (slide 5) |
| Slides 4–5 | Gráficos full-width com datalabels | 4 cards (2 donuts + 1 bar + 1 barras horizontais) |
| Subtítulo no header | **Ausente** | Presente |
| Padding dos slides | **16px 24px** | 24px 36px |
| KPI card padding | **12px 18px** | 22px 24px |
| KPI value size | **32px** | 42px |
| Table cell size | **15px** | 19px |
| Dots no footer | **5 (corretos)** | 6 (com duplicatas) |
| Plugin datalabels | **Presente** | Ausente |
| `buildChartsSlide6()` | Wrapper legado (1 declaração) | 3 declarações duplicadas |
