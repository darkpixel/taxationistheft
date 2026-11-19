# taxationistheft.tax

A single static page, `index.html`, served by nginx. It follows Bob's $10,000 invoice through every tax until the money lands in Bob's and Sue's personal bank accounts. The goal is to make taxation feel concrete, using numbers that hold up to scrutiny.

## The story (keep it consistent)

- **Bob's Computers** is a single-member LLC in Washington State with no S-corp election. Bob takes no paycheck: the LLC's profit is his personal income, and he pays self-employment tax and income tax through quarterly estimates (1040-ES). He takes the after-tax money as an owner's draw, which has no tax taken out.
- **Sue** is the one W-2 employee, at $5,000/month ($60k/yr, about 173 hrs/month).
- A client pays a **$10,000 check** for a month of IT support. The invoice did *not* list sales tax, so under WAC 458-20-107 the state treats $10,000 as the pre-tax price, and Bob owes the full ~10% out of pocket.
- Scope is **taxes only**, from the check to the two personal bank accounts. The user deliberately removed property tax, utilities, vehicles, insurance, groceries and other spending. Don't add them back unless asked. Two exceptions, both added at the user's request and kept out of the $10,000 totals: the property-levy example and the closing "This isn't the end" example of a single $150 restaurant dinner.

## Page structure

1. **Trunk (before the split):** sales tax $1,000 → B&O $47.10 (0.471% retailing rate; IT services are retail sales since ESSB 5814, Oct 1 2025) → split. The split checkpoint shows what Sue's job costs the company ($5,000 gross + $487.02 employer taxes = $5,487.02) and the $3,465.88 profit left; the script computes these from the row data. The employer taxes are still itemized, and counted in "Taxes so far", in stage 1.
2. **Two side-by-side branches, aligned by stage** (business on the left, Sue on the right):
   - Stage 1, payroll taxes. Business: employer Social Security, Medicare, FUTA, WA unemployment, L&I employer share. Sue: Social Security, Medicare, PFML, WA Cares, L&I employee share.
   - Stage 2, income taxes. Business: Bob's self-employment tax + federal income tax on the $3,465.88 profit. Sue: federal withholding.
   - End: into their personal bank accounts.
3. **Who has a safety net?** Two cards in the same two-column grid as the branches (Bob on the left, Sue on the right), with rows aligned across the cards. Rendered from the `safety` array in the script.
4. **Merged total:** a stacked bar plus a table by stage.
5. **Voting to steal from other people** (the user's title): the same two-column cards. Bob owns a paid-off $300,000 home on 2 acres; Sue rents an identical one from a landlord who owns it outright. An EMS levy ($0.50 per $1,000, $150/yr) and a school levy ($2.00 per $1,000, $600/yr) add $750/yr ($62.50/month) to Bob's bill and to the landlord's, while Sue gets the same vote and no bill. The landlord's row shows the only two choices: absorb the $750 a year or raise Sue's rent $62.50 a month. Bob homeschools his kids and owes the school levy anyway; Sue's kids attend the public school it funds. Rates are illustrative, within the legal caps. Rendered from the `levies` array.
6. **This isn't the end:** one $150 dinner (total paid, sales tax included, no tip): sales tax $13.64, restaurant B&O $0.64, and an estimated $11.20 in taxes on staff wages (30% of the $136.36 menu price, at Sue's rates), $25.48 in all. It then lists where the rest gets taxed again and ends with "Taxation is theft, and it never ends." The script computes the figures from the page's own rates; the wage line is labeled as an estimate.

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

- **Production is DigitalOcean App Platform**, as a static site defined in `.do/app.yaml`. It uses the plain-HTML buildpack (no build step) and redeploys on every push to `master`. Every file in the repo is served, so don't commit anything that shouldn't be public. The nginx security headers and `/healthz` below don't apply there.
- The Docker image is for running the page locally and for the earlier Kubernetes deployment:

- `Dockerfile`: `nginxinc/nginx-unprivileged` (non-root, uid 101) serving on port 8080, configured by `nginx.conf`, with `/healthz` for probes.
- `.github/workflows/dockerimage.yml` builds on every push to `ghcr.io/darkpixel/taxationistheft`, tagged with the branch, the short sha and `latest` on the default branch.
