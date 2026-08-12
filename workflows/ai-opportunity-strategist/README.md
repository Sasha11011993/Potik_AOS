# AI Opportunity Strategist — Content Ideas

Manual n8n workflow that turns real freelance-market demand into demonstration n8n automation ideas and Ukrainian social-media content packages for LinkedIn and Instagram.

## Purpose

The workflow does not publish posts automatically. It creates draft content for review:

- client problem and target audience;
- n8n automation/demo idea and solution architecture;
- potential service and technology-fit rationale;
- hook, LinkedIn post, Instagram post, CTA, and visual/carousel idea;
- demand, technology, portfolio, content, and overall opportunity scores.

## Data flow

`Start Content Strategy` → `Read Demand Report` → `Read Demand Analysis` → `Read Tech Signals` → `Build Content Briefs` → `Prepare OpenAI Request` → `Generate Content Opportunities` → `Shape Content Rows` → `Upsert Content Opportunities`

The workflow uses the top five clusters from `Demand Report (Звіт про попит)`, up to three related examples from `Demand Analysis (Аналіз попиту)`, and current records from `Tech_Signals (Технологічні сигнали)`.

For each cluster, OpenAI can return at most two ideas. A run can therefore create or update at most ten opportunities. If no relevant technology signal applies, the model may create a `demand-only` idea based on n8n core capabilities.

## Scores

All scores use the 0–100 range.

```text
opportunity_score = 45% market_demand_score
                  + 35% technology_fit_score
                  + 10% portfolio_potential
                  + 10% content_potential
```

The maximum `opportunity_score` is 100.

## Storage and deduplication

Results are written to `Content Opportunities (Контент-можливості)` with `appendOrUpdate`. The stable key is:

```text
CO::<demand_cluster>::<tech_signal_id-or-demand-only>::<idea_slug>
```

Running the workflow again updates the matching row instead of creating a duplicate. New records receive `status = draft` for manual review.

## Credentials and reliability

- Google Sheets OAuth2 credential: `Potik_AOS`
- OpenAI HTTP Bearer credential: `Potik_AOS-OpenAI-Bearer`
- Shared Error Workflow: `Обробник помилок`

Google Sheets and OpenAI nodes retry up to three times with a five-second interval. Credentials contain secrets; no secret is stored in the workflow definition.

## Operating mode

The workflow is intentionally manual and inactive. Do not publish it unless automatic execution is explicitly required.

## Validation

Live manual execution was verified after implementation: five demand clusters produced five OpenAI responses, ten content rows, and ten successful Google Sheets upserts.
