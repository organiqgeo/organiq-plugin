# Organiq plugin for Claude Code

[Português abaixo](#plugin-organiq-para-claude-code)

Organiq (organiqgeo.com) is a read-only MCP connector that syncs Google Search Console, GA4 sessions from AI
assistants and Bing data daily into its own database, keeps history beyond Google's 16 months, and exposes 39 analysis
tools to Claude, ChatGPT, Cursor and Gemini CLI. This repository is the Claude Code plugin: seven skills that
teach Claude how to analyse your own data through the connector, plus the connector entry (`.mcp.json`).

## Quick facts

| | |
|---|---|
| Connector | `https://organiqgeo.com/mcp` (remote, Streamable HTTP) |
| Tools | 39, all read-only; 3 work before sign-in (`get_capabilities`, `get_ctr_benchmark`, `check_connection`) |
| Authentication | OAuth 2.1 with Google sign-in; API key for clients without OAuth |
| Data | Search Console, GA4 sessions from ChatGPT, Perplexity, Gemini, Copilot and Claude, Bing Webmaster, Core Web Vitals |
| Needs | A site verified in Google Search Console. The Google sign-in creates the Organiq account and imports the properties; the dashboard is `https://app.organiqgeo.com` |
| License | Apache-2.0 |

## Install

**Claude Code (plugin, includes the skills):**

```bash
claude plugin marketplace add organiqgeo/organiq-plugin
claude plugin install organiq@organiq
```

Restart Claude Code (or run `/reload-plugins`), then `/mcp` → `organiq` → Authenticate and sign in with Google.
Connector only, without the skills: `claude mcp add --transport http organiq https://organiqgeo.com/mcp`.

**Claude (claude.ai):** Settings → Connectors → Add custom connector → name `Organiq`, URL
`https://organiqgeo.com/mcp`, leave Client ID and secret empty → Add → Connect → sign in with Google → Allow.

**Cursor:** add this under `mcpServers` in `~/.cursor/mcp.json` (or `.cursor/mcp.json` in the project), then
Settings → MCP → enable `organiq` and approve the sign-in window:

```json
{ "mcpServers": { "organiq": { "url": "https://organiqgeo.com/mcp" } } }
```

Step-by-step guides, also for ChatGPT and Gemini CLI: `https://organiqgeo.com/docs`.

## Tools (39, read-only)

- **Connection and discovery:** `get_capabilities`, `get_ctr_benchmark`, `check_connection`, `list_sites`
- **Performance:** `get_site_summary`, `get_timeseries`, `query_top_queries`, `query_top_pages`, `query_countries`, `query_devices`, `get_breakdown`, `breakdown_by_path`, `get_query_performance`, `get_page_performance`, `get_ranking_changes`, `get_position_distribution`, `get_branded_performance`, `get_longtail_clusters`, `get_annotations`
- **Opportunities:** `get_ctr_curve`, `find_decaying_content`, `analyze_cannibalization`, `score_opportunities`, `find_page_poaching_opportunities`
- **Statistics:** `detect_anomalies`, `detect_change_points`, `forecast_traffic`
- **AI and query shapes:** `get_ai_traffic`, `get_genai_impressions`, `analyze_query_shapes`, `get_query_shape_trend`, `analyze_sampling_impact`
- **Technical and Bing:** `get_core_web_vitals`, `get_indexing_tracker`, `list_url_inspections`, `list_sitemaps`, `get_bing_performance`
- **Reports:** `generate_seo_report`, `get_weekly_digest`

Every answer states the property, the period and the comparison used. CTR is a fraction (0.052 = 5.2%) and position
is an impression-weighted average (lower is better).

## Skills

| Skill | Use when |
|---|---|
| `quick-wins` | queries with CTR below expected, striking distance, what to optimise first |
| `content-decay` | pages losing clicks, refresh candidates, why a page dropped |
| `cannibalization` | several URLs ranking for the same query |
| `weekly-report` | weekly/monthly status for a site or client |
| `ai-traffic` | sessions from ChatGPT, Perplexity, Gemini, Copilot, Claude; conversational queries |
| `geo-audit` | monthly GEO report: AI referrals per source, question-style queries, AI Overviews hypothesis, 3 actions |
| `site-audit` | full one-pass audit of a property: headline, trajectory, mix, movers, decay, cannibalization, quick wins |

The same skills are published for any agent at `https://organiqgeo.com/.well-known/agent-skills/index.json`.

## Verify

```bash
claude plugin validate /path/to/organiq-plugin
claude plugin list                  # organiq@organiq enabled
claude mcp list                     # organiq … Connected (after /mcp → Authenticate)
```

In a session, `/mcp` shows the `organiq` server with 39 read-only tools, and typing "quick wins for example.com"
loads the `quick-wins` skill.

## Data and privacy

The skills send only tool arguments (property name, dates, filters) to the Organiq server. The server returns
aggregated Search Console and GA4 figures for properties in your own account, and never receives the conversation.
Nothing is written to Google. Disconnecting in the Google account settings revokes access.

[Privacy](https://organiqgeo.com/privacy) · [Terms](https://organiqgeo.com/terms) · [Support](https://organiqgeo.com/support) · [Security](https://organiqgeo.com/security) · [Docs](https://organiqgeo.com/docs)

## About

Organiq is operated by Youssef Pizolotto Umar, Brazil. Contact: youssefpmat@gmail.com. Contracting party and billing
details are in the Terms. License: Apache-2.0.

---

# Plugin Organiq para Claude Code

Organiq (organiqgeo.com) é um conector MCP somente leitura que sincroniza todo dia o Google Search Console, as sessões
do GA4 vindas de assistentes de IA e o Bing num banco próprio, guarda o histórico além dos 16 meses do Google e expõe
39 ferramentas de análise ao Claude, ChatGPT, Cursor e Gemini CLI. Este repositório é o plugin para Claude Code: sete
skills que ensinam o Claude a analisar os seus próprios dados pelo conector, mais a entrada do conector (`.mcp.json`).

## Fatos rápidos

| | |
|---|---|
| Conector | `https://organiqgeo.com/mcp` (remoto, Streamable HTTP) |
| Ferramentas | 39, todas somente leitura; 3 funcionam sem login (`get_capabilities`, `get_ctr_benchmark`, `check_connection`) |
| Autenticação | OAuth 2.1 com login Google; API key para clientes sem OAuth |
| Dados | Search Console, sessões do GA4 vindas de ChatGPT, Perplexity, Gemini, Copilot e Claude, Bing Webmaster, Core Web Vitals |
| Requisito | Um site verificado no Google Search Console. O login Google cria a conta Organiq e importa as propriedades; o painel é `https://app.organiqgeo.com` |
| Licença | Apache-2.0 |

## Instalar

**Claude Code (plugin, com as skills):**

```bash
claude plugin marketplace add organiqgeo/organiq-plugin
claude plugin install organiq@organiq
```

Reinicie o Claude Code (ou rode `/reload-plugins`), depois `/mcp` → `organiq` → Authenticate e entre com o Google.
Só o conector, sem as skills: `claude mcp add --transport http organiq https://organiqgeo.com/mcp`.

**Claude (claude.ai):** Configurações → Conectores → Adicionar conector personalizado → nome `Organiq`, URL
`https://organiqgeo.com/mcp`, client ID e secret em branco → Adicionar → Conectar → login Google → Permitir.

**Cursor:** adicione isto em `mcpServers` no `~/.cursor/mcp.json` (ou `.cursor/mcp.json` no projeto), depois
Configurações → MCP → ative `organiq` e aprove a janela de login:

```json
{ "mcpServers": { "organiq": { "url": "https://organiqgeo.com/mcp" } } }
```

Guias passo a passo, também para ChatGPT e Gemini CLI: `https://organiqgeo.com/pt/docs`.

## Ferramentas (39, somente leitura)

- **Conexão e descoberta:** `get_capabilities`, `get_ctr_benchmark`, `check_connection`, `list_sites`
- **Desempenho:** `get_site_summary`, `get_timeseries`, `query_top_queries`, `query_top_pages`, `query_countries`, `query_devices`, `get_breakdown`, `breakdown_by_path`, `get_query_performance`, `get_page_performance`, `get_ranking_changes`, `get_position_distribution`, `get_branded_performance`, `get_longtail_clusters`, `get_annotations`
- **Oportunidades:** `get_ctr_curve`, `find_decaying_content`, `analyze_cannibalization`, `score_opportunities`, `find_page_poaching_opportunities`
- **Estatística:** `detect_anomalies`, `detect_change_points`, `forecast_traffic`
- **IA e formatos de consulta:** `get_ai_traffic`, `get_genai_impressions`, `analyze_query_shapes`, `get_query_shape_trend`, `analyze_sampling_impact`
- **Técnico e Bing:** `get_core_web_vitals`, `get_indexing_tracker`, `list_url_inspections`, `list_sitemaps`, `get_bing_performance`
- **Relatórios:** `generate_seo_report`, `get_weekly_digest`

Toda resposta declara a propriedade, o período e a comparação usados. CTR é fração (0,052 = 5,2 %) e posição é média
ponderada por impressões (menor é melhor).

## Skills

| Skill | Use quando |
|---|---|
| `quick-wins` | consultas com CTR abaixo do esperado, striking distance, o que otimizar primeiro |
| `content-decay` | páginas perdendo cliques, candidatas a atualização, por que uma página caiu |
| `cannibalization` | várias URLs ranqueando para a mesma consulta |
| `weekly-report` | status semanal/mensal de um site ou cliente |
| `ai-traffic` | sessões vindas de ChatGPT, Perplexity, Gemini, Copilot, Claude; consultas conversacionais |
| `geo-audit` | relatório GEO mensal: referências de IA por origem, consultas em forma de pergunta, hipótese de Visão geral de IA, 3 ações |
| `site-audit` | auditoria completa de uma propriedade numa passada: números, trajetória, mix, movimentos, decaimento, canibalização, quick wins |

As mesmas skills estão publicadas para qualquer agente em `https://organiqgeo.com/.well-known/agent-skills/index.json`.

## Verificar

```bash
claude plugin validate /caminho/para/organiq-plugin
claude plugin list                  # organiq@organiq habilitado
claude mcp list                     # organiq … Connected (após /mcp → Authenticate)
```

Numa sessão, `/mcp` mostra o servidor `organiq` com 39 ferramentas somente leitura, e escrever "quick wins do
exemplo.com.br" carrega a skill `quick-wins`.

## Dados e privacidade

As skills enviam ao servidor Organiq só os argumentos das ferramentas (nome da propriedade, datas, filtros). O
servidor devolve números agregados do Search Console e do GA4 das propriedades da sua própria conta e nunca recebe a
conversa. Nada é escrito no Google. Desconectar nas configurações da conta Google revoga o acesso.

[Privacidade](https://organiqgeo.com/pt/privacy) · [Termos](https://organiqgeo.com/pt/terms) · [Suporte](https://organiqgeo.com/pt/support) · [Segurança](https://organiqgeo.com/pt/security) · [Docs](https://organiqgeo.com/pt/docs)

## Sobre

O Organiq é operado por Youssef Pizolotto Umar, Brasil. Contato: youssefpmat@gmail.com. Parte contratante e dados de
cobrança estão nos Termos. Licença: Apache-2.0.
