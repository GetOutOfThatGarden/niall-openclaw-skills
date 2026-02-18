---
name: revenue-path-writer
description: Create comprehensive, investor-ready revenue path reports for startups and projects. Generates B2B SaaS monetization strategies with financial projections, market analysis, and go-to-market plans. Use when the user needs a revenue model document, business plan for investors, monetization strategy, or path-to-profitability analysis. Produces clean markdown output suitable for AI agents and human investors.
---

# Revenue Path Writer

This skill teaches AI agents to create professional revenue path reports for startups, hackathon projects, and business ventures.

## Purpose

Generate investor-ready documents that demonstrate:
- Clear path to revenue (not just ideas)
- Market opportunity and competitive positioning
- Financial projections with unit economics
- Go-to-market strategy
- Risk mitigation

## Output Format

Always produce clean markdown with:
- Executive summary at top
- Numbered sections (1–9)
- Tables for data
- No sensitive information (API keys, passwords, tokens)
- Generic audience (not specific to one investor)

## Required Sections

### 1. Executive Summary
- What the project does (1–2 sentences)
- Revenue model (B2B SaaS, API fees, etc.)
- Time to revenue
- Projected ARR (Year 1, Year 3)

### 2. The Problem (Revenue-Generating Pain Point)
- Current costs customers face
- Comparison table (Traditional vs Your Solution)
- Customer willingness to pay

### 3. Revenue Streams (At least 2)
- Primary stream (API usage, subscriptions, etc.)
- Secondary streams (consulting, white-label, enterprise)
- Pricing tiers with numbers

### 4. Go-to-Market Strategy
- Phase 1: Beta/Pilot (Months 1–3)
- Phase 2: Paid Pilots (Months 4–6)
- Phase 3: Scale (Months 7–12)

### 5. Market Size
- TAM (Total Addressable Market)
- SAM (Serviceable Addressable Market)
- SOM (Serviceable Obtainable Market)

### 6. Competitive Positioning
- vs. Traditional competitors (table format)
- vs. Other blockchain/web3 solutions
- Clear value proposition

### 7. Financial Projections
- Year 1 quarterly breakdown
- Year 3 targets
- Unit economics (CAC, LTV, LTV:CAC ratio)

### 8. Risk Mitigation
- Regulatory risk + mitigation
- Technical risk + mitigation
- Market risk + mitigation
- Competition risk + mitigation

### 9. Traction & Validation
- Current status (MVP, deployed, etc.)
- Hackathon submissions / validation
- Next milestones (dated)

### 10. Why [Project] Wins
- 5 key differentiators
- Keep concise, punchy

### Conclusion
- Bullet summary of strengths
- No "investment ask" section (keep it generic)

## Security Rules (CRITICAL)

**NEVER include:**
- API keys, tokens, passwords
- Telegram handles or personal contact info
- Specific investor names or programs
- Server IPs or infrastructure details
- Private repository URLs

**Always include:**
- GitHub repo URL (public)
- Demo video URL (Loom, etc.)
- Generic contact method (GitHub issues, etc.)

## Example Workflow

```
User: "Create a revenue path report for my DeFi analytics tool"

Agent should:
1. Ask: Target customers? Pricing thoughts? Current traction?
2. Research market size for DeFi analytics
3. Create document following this structure
4. Review for sensitive data
5. Present for feedback
6. Iterate based on user input
```

## Best Practices

1. **Use real numbers** — Don't guess market size, research it
2. **Conservative projections** — Better to under-promise
3. **Specific milestones** — "March 2026" not "next quarter"
4. **Multiple revenue streams** — Shows diversification
5. **Acknowledge risks** — Shows maturity
6. **Clean markdown** — Readable by AI agents and humans

## Common Mistakes to Avoid

- ❌ Vague claims ("we'll make millions")
- ❌ No unit economics
- ❌ Missing competitive analysis
- ❌ Unrealistic timelines
- ❌ Including sensitive data
- ❌ Targeting only one investor

## References

Based on patterns from:
- MinKYC Revenue Path (B2B SaaS, API pricing)
- Standard YC/VC pitch formats
- OpenClaw skill-creator guidelines

---

*Version 1.0 — Created by Niall*
