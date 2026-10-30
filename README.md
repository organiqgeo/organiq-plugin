# Organiq plugin for Claude Code

[Português abaixo](#plugin-organiq-para-claude-code)

Organiq (organiqgeo.com) is an MCP connector with read-only Google access that syncs Google Search Console, GA4 sessions from AI assistants and Bing data daily into its own database, keeps history beyond Google's 16 months, and exposes 44 analysis and setup tools to Claude, ChatGPT, Cursor and Gemini CLI. Free plan; paid plans from US$ 9/month. Operated by Youssef Pizolotto Umar.

This repository is the Claude Code plugin: seven skills that teach Claude how to analyse your own data through the connector, plus the connector entry (`.mcp.json`).

## Quick facts

| | |
|---|---|
| Connector | `https://organiqgeo.com/mcp` (remote, Streamable HTTP) |
| Tools | 44: 39 read tools and 5 setup tools; 3 work before sign-in (`get_capabilities`, `get_ctr_benchmark`, `check_connection`) |
| Google access | Read-only. The read tools only read; the setup tools write only Organiq settings (linked GA4, Bing site and key, brand terms) and need the `mcp:write` scope. Nothing is written to Search Console, GA4 or Bing |
| Authentication | OAuth 2.1 with Google sign-in; API key for clients without OAuth |
| Data | Search Console, GA4 sessions from ChatGPT, Perplexity, Gemini, Copilot and Claude, Bing Webmaster, Core Web Vitals |
| Needs | A site verified in Google Search Console. The Google sign-in creates the Organiq account and imports the properties; the dashboard is `https://app.organiqgeo.com` |
| License | Apache-2.0 |

## How Organiq differs, with the exception for each

- Remote connector with Google sign-in: no Google Cloud project, service account or downloaded key. Exception: GSC Wizard, Ahrefs MCP, Semrush MCP and mcpsearchconsole.com are also remote with sign-in, and AminForou/mcp-gsc has a hosted version; the five open-source servers in the table run locally with Google Cloud credentials.
- No tool writes to Search Console, GA4 or Bing. The 4 setup tools that write (set_bing_key, set_bing_site, set_branded_terms, set_ga4_property) change Organiq settings and nothing else, and need the mcp:write scope. Exception: ahonn/mcp-server-gsc, Ahrefs MCP, Semrush MCP and mcpsearchconsole.com do not write either, and GSC Wizard can be connected read-only.
- A tool for Google's generative-AI performance report (AI Overviews and AI Mode impressions): get_genai_impressions. Exception: the data enters through the CSV exported from Search Console, the tool needs the GEO add-on (included in Agency), and the tool lists of Suganthans-GSC-MCP, gsc-mcp-full and mcpsearchconsole.com were not checked.
- Permanent free plan with the MCP connector: 1 property and 90 days of history. Exception: mcpsearchconsole.com also has a free plan (3 properties, 15 data pulls a day), and Suganthans-GSC-MCP and gsc-mcp-full are free open-source servers that run locally.
- Prices in Brazilian reais (BRL) as well as US dollars (USD): Starter R$ 49 and Agency R$ 249 a month; Starter US$ 9 and Agency US$ 49. Exception: among the alternatives, the currency was compared with GSC Wizard (selector with euro, US dollar, pound sterling and Indian rupee) and Ahrefs (dollar); mcpsearchconsole.com charges in euro (Solo €4.99 a month).

Source of each exception (one link per cell, read on October 30, 2026): https://organiqgeo.com/compare

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

## Tools (44: 39 read, 5 setup)

- **Connection and discovery:** `get_capabilities`, `get_ctr_benchmark`, `check_connection`, `list_sites`
- **Performance:** `get_site_summary`, `get_timeseries`, `query_top_queries`, `query_top_pages`, `query_countries`, `query_devices`, `get_breakdown`, `breakdown_by_path`, `get_query_performance`, `get_page_performance`, `get_ranking_changes`, `get_position_distribution`, `get_branded_performance`, `get_longtail_clusters`, `get_annotations`
- **Opportunities:** `get_ctr_curve`, `find_decaying_content`, `analyze_cannibalization`, `score_opportunities`, `find_page_poaching_opportunities`
- **Statistics:** `detect_anomalies`, `detect_change_points`, `forecast_traffic`
- **AI and query shapes:** `get_ai_traffic`, `get_genai_impressions`, `analyze_query_shapes`, `get_query_shape_trend`, `analyze_sampling_impact`
- **Technical and Bing:** `get_core_web_vitals`, `get_indexing_tracker`, `list_url_inspections`, `list_sitemaps`, `get_bing_performance`
- **Reports:** `generate_seo_report`, `get_weekly_digest`
- **Setup (scope `mcp:write`):** `list_ga4_properties`, `set_ga4_property`, `set_bing_site`, `set_bing_key`, `set_branded_terms`. They link GA4, set the Bing site and API key and replace the brand terms; `list_ga4_properties` only lists. The Bing API key is generated by you in the Bing portal and pasted in the chat. The sign-in asks for `mcp:read mcp:write`; if you connected before, remove the connector and add it again.

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
| `site-audit` | one-pass audit of a property: headline, trajectory, mix, movers, decay, cannibalization, quick wins |

The same skills are published for any agent at `https://organiqgeo.com/.well-known/agent-skills/index.json`.

## Verify

```bash
claude plugin validate /path/to/organiq-plugin
claude plugin list                  # organiq@organiq enabled
claude mcp list                     # organiq … Connected (after /mcp → Authenticate)
```

In a session, `/mcp` shows the `organiq` server with 44 tools (39 read, 5 setup), and typing "quick wins for example.com"
loads the `quick-wins` skill.

## Data and privacy

The skills send only tool arguments (property name, dates, filters) to the Organiq server. The server returns
aggregated Search Console and GA4 figures for properties in your own account, and never receives the conversation.
Nothing is written to Google or Bing; the setup tools write only Organiq settings. Disconnecting in the Google account settings revokes access.

[Privacy](https://organiqgeo.com/privacy) · [Terms](https://organiqgeo.com/terms) · [Support](https://organiqgeo.com/support) · [Security](https://organiqgeo.com/security) · [Docs](https://organiqgeo.com/docs)

## About

Organiq is operated by Youssef Pizolotto Umar, Brazil. Contact: youssefpmat@gmail.com. Contracting party and billing
details are in the Terms. License: Apache-2.0.

---

# Plugin Organiq para Claude Code

Organiq (organiqgeo.com) é um conector MCP com acesso somente leitura ao Google que sincroniza todo dia Search Console, sessões de assistentes de IA no GA4 e Bing num banco próprio, guarda histórico além dos 16 meses do Google e expõe 44 ferramentas ao Claude, ChatGPT, Cursor e Gemini CLI. Plano grátis; pagos desde R$ 49/mês. Operado por Youssef Pizolotto Umar.

Este repositório é o plugin para Claude Code: sete skills que ensinam o Claude a analisar os seus próprios dados pelo conector, mais a entrada do conector (`.mcp.json`).

## Fatos rápidos

| | |
|---|---|
| Conector | `https://organiqgeo.com/mcp` (remoto, Streamable HTTP) |
| Ferramentas | 44: 39 de leitura e 5 de setup; 3 funcionam sem login (`get_capabilities`, `get_ctr_benchmark`, `check_connection`) |
| Acesso ao Google | Somente leitura. As ferramentas de leitura só leem; as de setup gravam só configurações do Organiq (GA4 ligado, site e chave do Bing, termos de marca) e exigem o escopo `mcp:write`. Nada é escrito no Search Console, no GA4 nem no Bing |
| Autenticação | OAuth 2.1 com login Google; API key para clientes sem OAuth |
| Dados | Search Console, sessões do GA4 vindas de ChatGPT, Perplexity, Gemini, Copilot e Claude, Bing Webmaster, Core Web Vitals |
| Requisito | Um site verificado no Google Search Console. O login Google cria a conta Organiq e importa as propriedades; o painel é `https://app.organiqgeo.com` |
| Licença | Apache-2.0 |

## O que muda no Organiq, com a exceção de cada item

- Conector remoto com login Google: sem projeto no Google Cloud, conta de serviço ou chave baixada. Exceção: GSC Wizard, Ahrefs MCP, Semrush MCP e mcpsearchconsole.com também são remotos com login, e o AminForou/mcp-gsc tem versão hospedada; os cinco servidores de código aberto da tabela rodam localmente com credenciais do Google Cloud.
- Nenhuma ferramenta escreve no Search Console, no GA4 ou no Bing. As 4 ferramentas de setup que gravam (set_bing_key, set_bing_site, set_branded_terms, set_ga4_property) mudam só configurações do Organiq e exigem o escopo mcp:write. Exceção: ahonn/mcp-server-gsc, Ahrefs MCP, Semrush MCP e mcpsearchconsole.com também não escrevem, e o GSC Wizard pode ser conectado só para leitura.
- Ferramenta para o relatório de desempenho de IA generativa do Google (impressões na Visão geral de IA e no Modo IA): get_genai_impressions. Exceção: os dados entram pelo CSV exportado do Search Console, a ferramenta exige o add-on GEO (incluído na Agência), e as listas de ferramentas de Suganthans-GSC-MCP, gsc-mcp-full e mcpsearchconsole.com não foram conferidas.
- Plano grátis permanente com o conector MCP: 1 propriedade e 90 dias de histórico. Exceção: o mcpsearchconsole.com também tem plano grátis (3 propriedades, 15 consultas de dados por dia), e Suganthans-GSC-MCP e gsc-mcp-full são servidores de código aberto gratuitos que rodam localmente.
- Preço em reais (BRL) além de dólares (USD): Starter R$ 49 e Agência R$ 249 por mês; Starter US$ 9 e Agência US$ 49. Exceção: entre as alternativas, a moeda foi comparada com o GSC Wizard (seletor com euro, dólar americano, libra esterlina e rupia indiana) e o Ahrefs (dólar); o mcpsearchconsole.com cobra em euro (Solo €4,99 por mês).

Fonte de cada exceção (um link por célula, lido em 30 de outubro de 2026): https://organiqgeo.com/pt/compare

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

## Ferramentas (44: 39 de leitura, 5 de setup)

- **Conexão e descoberta:** `get_capabilities`, `get_ctr_benchmark`, `check_connection`, `list_sites`
- **Desempenho:** `get_site_summary`, `get_timeseries`, `query_top_queries`, `query_top_pages`, `query_countries`, `query_devices`, `get_breakdown`, `breakdown_by_path`, `get_query_performance`, `get_page_performance`, `get_ranking_changes`, `get_position_distribution`, `get_branded_performance`, `get_longtail_clusters`, `get_annotations`
- **Oportunidades:** `get_ctr_curve`, `find_decaying_content`, `analyze_cannibalization`, `score_opportunities`, `find_page_poaching_opportunities`
- **Estatística:** `detect_anomalies`, `detect_change_points`, `forecast_traffic`
- **IA e formatos de consulta:** `get_ai_traffic`, `get_genai_impressions`, `analyze_query_shapes`, `get_query_shape_trend`, `analyze_sampling_impact`
- **Técnico e Bing:** `get_core_web_vitals`, `get_indexing_tracker`, `list_url_inspections`, `list_sitemaps`, `get_bing_performance`
- **Relatórios:** `generate_seo_report`, `get_weekly_digest`
- **Setup (escopo `mcp:write`):** `list_ga4_properties`, `set_ga4_property`, `set_bing_site`, `set_bing_key`, `set_branded_terms`. Elas ligam o GA4, definem o site e a chave da API do Bing e substituem os termos de marca; `list_ga4_properties` só lista. A chave da API do Bing é gerada por você no portal do Bing e colada no chat. O login pede `mcp:read mcp:write`; se você conectou antes, remova o conector e adicione de novo.

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
| `site-audit` | auditoria de uma propriedade numa passada: números, trajetória, mix, movimentos, decaimento, canibalização, quick wins |

As mesmas skills estão publicadas para qualquer agente em `https://organiqgeo.com/.well-known/agent-skills/index.json`.

## Verificar

```bash
claude plugin validate /caminho/para/organiq-plugin
claude plugin list                  # organiq@organiq habilitado
claude mcp list                     # organiq … Connected (após /mcp → Authenticate)
```

Numa sessão, `/mcp` mostra o servidor `organiq` com 44 ferramentas (39 de leitura, 5 de setup), e escrever "quick wins do
exemplo.com.br" carrega a skill `quick-wins`.

## Dados e privacidade

As skills enviam ao servidor Organiq só os argumentos das ferramentas (nome da propriedade, datas, filtros). O
servidor devolve números agregados do Search Console e do GA4 das propriedades da sua própria conta e nunca recebe a
conversa. Nada é escrito no Google nem no Bing; as ferramentas de setup gravam só configurações do Organiq. Desconectar nas configurações da conta Google revoga o acesso.

[Privacidade](https://organiqgeo.com/pt/privacy) · [Termos](https://organiqgeo.com/pt/terms) · [Suporte](https://organiqgeo.com/pt/support) · [Segurança](https://organiqgeo.com/pt/security) · [Docs](https://organiqgeo.com/pt/docs)

## Sobre

O Organiq é operado por Youssef Pizolotto Umar, Brasil. Contato: youssefpmat@gmail.com. Parte contratante e dados de
cobrança estão nos Termos. Licença: Apache-2.0.
