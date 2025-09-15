Insight
The most important finding is that the portfolio’s default rate rises steadily from grade A through G, and that 60-month loans show materially higher default rates than 36-month loans within the same grade. This is consistent with LendingClub’s grading system, where lower grades encode higher probability of default.

Actionable recommendation
Implement targeted credit-policy and pricing adjustments for lower grades and especially for 60-month terms. Examples include raising minimum income thresholds, tightening maximum DTI, or setting higher rate floors. Then monitor post-change default rates by grade×term in a lightweight dashboard built from the Part 1 view. This will help bring expected losses back within portfolio targets by aligning eligibility and price with observed risk segmentation, rather than relying only on headline rates to compensate for elevated defaults.

Follow-up
A likely stakeholder question is whether the higher interest rates offered to lower grades and longer terms actually compensate for their higher default risk on a net, risk-adjusted basis after expected losses and fees. To answer this, I would prepare a cohort-level expected return analysis by grade×term, using the formula:

expected return = coupon yield − (default rate × LGD) − servicing/origination fees

This would use the same 2007–2015 dataset and default definition, supplemented by recovery/LGD data if available or a documented LGD assumption.
