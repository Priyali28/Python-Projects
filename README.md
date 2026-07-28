# GA4 E-Commerce Funnel & Conversion Analysis

## Objective
Analyze user behavior on Google's public GA4 e-commerce dataset (Google Merchandise Store) to identify where users drop off in the purchase funnel, and whether conversion rate differences across device types, traffic sources, and days of the week are statistically meaningful — moving from surface-level observation to a defensible, data-driven business recommendation.

## Dataset
`bigquery-public-data.ga4_obfuscated_sample_ecommerce` — event-level GA4 export data (November 2020), queried via Google Cloud BigQuery.

## Tools
- Google BigQuery (SQL)
- Python: Pandas, NumPy, Matplotlib, statsmodels
- Google Colab

## Methodology

**1. Funnel Analysis**
Aggregated unique users at each stage of the purchase funnel (session start → view item → add to cart → begin checkout → purchase) to identify overall conversion volume and calculate stage-to-stage drop-off rates.

**2. Traffic Source Performance**
Compared user volume and conversion rate across acquisition channels (Google, direct, referral, branded domain traffic, etc.) to identify which sources drive the most *efficient* conversions, not just the most volume.

**3. Device Category Analysis**
Compared conversion rates across mobile, desktop, and tablet users.

**4. Statistical Significance Testing**
Applied two-proportion z-tests (statsmodels) to validate whether observed conversion-rate differences — across traffic sources and across device categories — were statistically significant or attributable to random variation.

**5. Cross-Segment Analysis**
Broke down device-level conversion rates by traffic source to test whether the device effect was consistent across channels or concentrated in specific ones.

**6. Day-of-Week Analysis**
Compared both raw purchase volume and user-normalized conversion rate by day of week, to distinguish genuine efficiency patterns from simple traffic-volume effects.

## Key Findings

**Funnel & Drop-off**
- Of 92,322 sessions, only 1,601 (1.7%) resulted in a purchase.
- The largest drop-off occurs between viewing an item and adding to cart (91.3% drop-off) — the single biggest leak in the funnel.
- **Data limitation identified:** `begin_checkout` event counts (4,474) exceeded `add_to_cart` counts (2,122) in the raw funnel, producing a nonsensical negative "drop-off" if treated as strictly sequential. This indicates the funnel is not purely linear in this dataset — some users reach checkout without a distinct add-to-cart event (e.g., "buy now" flows, or a cart action occurring in an earlier untracked session). This was flagged as a data/methodology caveat rather than smoothed over.

**Traffic Source**
- Google (34,388 users) and "\<Other\>" (26,582 users) drive the highest volume but convert at only 1.67% and 1.86% respectively.
- The branded domain (shop.googlemerchandisestore.com) converts notably higher (2.94%) despite far lower volume (8,448 users) — a smaller but more efficient channel.
- The difference between the top two traffic sources' conversion rates was **not statistically significant** (p = 0.0728, two-proportion z-test) — illustrating that an observed gap in a table doesn't always reflect a real underlying effect.

**Device Category**
- Mobile converts at a statistically significantly higher rate than desktop (2.74% vs. 2.47%, **p = 0.0194**, two-proportion z-test), despite desktop driving substantially higher traffic volume (46,310 vs. 31,824 users).
- Tablet is a small, low-converting segment (2.12%, only 1,789 users) with limited strategic weight given its size.

**Device × Traffic Source (Cross-Segment)**
- The mobile conversion advantage **holds across most traffic sources**, confirming it's a broad pattern rather than an artifact of one channel.
- The advantage is **most pronounced for Google-sourced traffic** (2.0% mobile vs. 1.5% desktop).
- One segment ("(data deleted)" — a GA4 privacy-redaction placeholder, not an identifiable channel) showed the reverse pattern; this is noted as a data limitation rather than an interpretable finding.

**Day of Week**
- Raw purchase counts peak on Monday (452) and Friday (352), with Sunday lowest (178).
- After normalizing by unique users, Friday (2.83%) converts essentially on par with Monday (2.80%) despite lower raw volume — while Tuesday's high raw count (358 purchases) reflects mostly higher traffic volume rather than higher efficiency (2.41% conversion rate, notably below Monday and Friday).
- Sunday is the weakest day on both raw volume and conversion rate.
- This reinforces a consistent theme across the analysis: raw counts and rates can tell different stories, and conclusions should be based on rate wherever segment sizes differ meaningfully.

## Business Recommendations
1. **Prioritize the view_item → add_to_cart transition** for UX investigation — it's the single largest leak in the funnel and the highest-leverage point for improvement.
2. **Reallocate incremental spend toward mobile-optimized experiences for Google-driven traffic specifically**, where the mobile conversion advantage is largest and most consistent, rather than applying a blanket device-level initiative.
3. **Investigate the branded-domain channel's outperformance** (shop.googlemerchandisestore.com) — understanding what drives its higher efficiency could inform improvements to lower-converting, higher-volume channels like Google and direct.
4. **Re-test traffic-source-level conversion differences with a larger sample window** before making acquisition budget decisions, given the borderline (non-significant) result observed here.
5. **Treat Monday and Friday as equally high-value days** for promotional timing based on conversion rate, rather than defaulting to Monday based on raw purchase volume alone.

## Limitations
- Data reflects a single month (November 2020) and may not generalize to other seasons or periods.
- Correlational analysis throughout — device, source, and day-of-week differences are associated with conversion outcomes, not proven causal drivers.
- The purchase funnel is not strictly sequential in this dataset; stage-to-stage drop-off figures should be interpreted directionally, not as precise counts of users lost between exact steps.
- Some traffic-source data is redacted by GA4 for privacy ("(data deleted)"), limiting full attribution for that segment.

## Future Work
- Extend significance testing with a Bayesian approach (posterior probability of one segment outperforming another, rather than a binary significance threshold).
- Segment analysis further by product category and campaign.
- Test for seasonality or novelty effects using a longer date range.
- Investigate the non-sequential funnel behavior further (e.g., using `event_params` or session-level event ordering) to better understand the begin_checkout/add_to_cart discrepancy.
