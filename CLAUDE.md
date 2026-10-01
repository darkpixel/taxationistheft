# taxationistheft.tax

A single static page, `index.html`, served by nginx. It follows Bob's $10,000 invoice through every tax until the money lands in Bob's and Sue's personal bank accounts. The goal is to make taxation feel concrete, using numbers that hold up to scrutiny.

## The story (keep it consistent)

- **Bob's Computers** is a single-member LLC in Washington State with no S-corp election. Bob takes no paycheck: the LLC's profit is his personal income, and he pays self-employment tax and income tax through quarterly estimates (1040-ES). He takes the after-tax money as an owner's draw, which has no tax taken out.
- **Sue** is the one W-2 employee, at $5,000/month ($60k/yr, about 173 hrs/month).
- A client pays a **$10,000 check** for a month of IT support. The invoice did *not* list sales tax, so under WAC 458-20-107 the state treats $10,000 as the pre-tax price, and Bob owes the full ~10% out of pocket.
- Scope is **taxes only**, from the check to the two personal bank accounts. The user deliberately removed property tax, utilities, vehicles, insurance, groceries and other spending. Don't add them back unless asked.

## Page structure

1. **Trunk (before the split):** sales tax $1,000 → B&O $47.10 (0.471% retailing rate; IT services are retail sales since ESSB 5814, Oct 1 2025) → split.
2. **Two side-by-side branches, aligned by stage** (business on the left, Sue on the right):
   - Stage 1, payroll taxes. Business: employer Social Security, Medicare, FUTA, WA unemployment, L&I employer share. Sue: Social Security, Medicare, PFML, WA Cares, L&I employee share.
   - Stage 2, income taxes. Business: Bob's self-employment tax + federal income tax on the $3,465.88 profit. Sue: federal withholding.
   - End: into their personal bank accounts.
3. **Merged total:** a stacked bar plus a table by stage.

Bar colors: orange = taxes, purple = Sue's money (her $5,000 paycheck and what she keeps), green = left with the business/Bob. Business-side bars always show the whole $10,000, and the business's "Taxes so far" includes the $1,047.10 taken before the split.

## Current numbers (2026, single filer, standard deduction $16,100)

| | Business → Bob | Sue |
|---|---|---|
| Before split | $1,047.10 | — |
| Payroll taxes | $487.02 | $464.34 |
| Income taxes | $649.46 (SE $489.71 + income $159.75) | $418.33 |
| Lands in bank account | **$2,816.42** | **$4,117.33** |

Total taxes $3,066.25 (30.7%). Everything must add back to exactly $10,000. The page's script computes the running totals and the final figures from the row data, so change amounts in the data arrays, not in display text. Some prose does repeat figures (e.g. "$3,465.88", "$5,487.02", "$1,047.10"), so update those when the inputs change.

Assumptions:
- ~10% combined sales tax and no city B&O.
- WA unemployment at 1.2% + 0.03%.
- L&I at about $0.30/hr, with the worker paying about 24%.
- PFML 1.13% total; the employee share is 71.43% (0.807%), and there's no employer share under 50 employees.
- WA Cares 0.58%.
- Social Security wage base $184,500.
- 2026 brackets: 10% to $12,400, 12% to $50,400.
- The 20% QBI deduction applies to Bob.

The WA unemployment and L&I rates are illustrative; they vary by employer.

## Writing and design rules

- Plain, specific copy. Payroll wording the user approved: the law requires the company to hold back the taxes on the front of Sue's check and pay them to the government on her behalf, and the company also pays additional taxes that never appear on her check.
- Keep numbers defensible. Label anything estimated, and keep fees-for-service separate from taxes.
- Colors are CSS tokens on `:root`, with dark mode under `prefers-color-scheme` and `[data-theme]`. Fonts are Public Sans and IBM Plex Mono from Google Fonts. The page must work at phone width with no horizontal scroll.
- No AI attribution in commits.

## Build and deploy

- `Dockerfile`: `nginxinc/nginx-unprivileged` (non-root, uid 101) serving on port 8080, configured by `nginx.conf`, with `/healthz` for probes.
- `.github/workflows/dockerimage.yml` builds on every push to `ghcr.io/darkpixel/taxationistheft`, tagged with the branch, the short sha and `latest` on the default branch.
