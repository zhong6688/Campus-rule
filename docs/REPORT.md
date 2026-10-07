# Beauty Helper: Business and Technical Trade-off Analysis

## 1. Problem and why it matters

First-time buyers of skincare, and people buying it as a gift, face thousands of products and reviews they cannot judge. Singapore marketplace reviews are also noisy: of the 114 negative reviews I collected, only about a third (40) described the product; the rest complained about delivery, packaging, the seller or price. Star ratings therefore say little about whether a moisturizer suits a skin type. Today a shopper reads those ratings and the first page of reviews, which are mostly positive and often about the parcel.

Beauty Helper takes a skin type and a budget and recommends one face moisturizer from a small knowledge base (7 products, S$5 to S$42 on the day I saw the prices), with real review quotes behind it, so the shopper can trace every claim. It declines what it cannot support: anti-ageing, sunscreen, makeup, treatment claims.

The likely buyer is a marketplace, retailer or brand that wants fewer wrong purchases and returns. I measured neither conversion nor returns, so this is a hypothesis; the prototype shows only that the pipeline can be built and checked.

## 2. System

Own code reads the message, answers some requests itself (missing skin type, out of scope, no product within budget), and keeps products within budget whose page lists the skin type. Claude picks one of at most three candidates and writes the reason and quotes. Own code then checks the answer: the pick must be a candidate within budget, every quote must appear word for word in the picked product's reviews, and sentences with treatment claims or unstated properties (for example "fragrance-free") are removed. A scikit-learn classifier tags reviews without a hand label and runs in the page.

## 3. Technical trade-offs

| Decision | Chosen | Alternative | What it costs |
| --- | --- | --- | --- |
| Platform | Claude plus own code | Dify or Coze with GPT-4o-mini (my Week 3 plan) | Runs only inside Claude, on the viewer's account; I did not compare models |
| Safety | Checks in code | Prompt instructions only | More code; but fabricated quotes can be tested for |
| Retrieval | Budget, skin type, keyword overlap | Vector search | Misses paraphrases; adequate for 7 products |
| Sentiment | Own scikit-learn model | Ask the LLM | Weaker on new products (76%); an LLM would probably do better but is slower and costs per review; not tested |
| Skin type | Types listed on the page | One label chosen by me | "All skin types" matches nothing specific |

## 4. Business trade-offs

**Who pays.** The page uses the viewer's own Claude account: no server cost for me, but a marker needs an account. A deployment needs a backend and an API budget. Each recommendation sends roughly 2,200 to 3,100 input tokens (mean about 2,700; an estimate, not billed counts) and returns a short JSON answer in 5 to 8 seconds. Five of the 14 scenario questions are answered by code at no cost, and the classifier runs in the browser. I did not look up a price, so I give no dollar figure.

**Liability.** Recommending a wrong product can irritate skin. The answer carries a patch-test line, never claims a treatment, and shows a doctor or pharmacist notice when a condition such as eczema is mentioned.

**Data.** Prices are sale prices from one day (for example S$9.52 against a regular S$30) and change daily. I copied data by hand; a real service would need a permitted feed from the marketplace.

**Neutrality.** If sellers pay for placement, recommendations could be biased. Verified quotes make answers auditable but do not guarantee neutrality.

## 5. Results and critique

| Assistant metric (14 scenario questions) | Target | Reached |
| --- | --- | --- |
| Pass rate | at least 80% | 14 of 14 |
| Recommendations with a verified quote | at least 90% | 9 of 9 |
| Quotes rejected as not verbatim | at most 10% | 0 of 24 |
| Average time per model call | at most 20 s | 5.9 s |

Classifier (121 reviews): hold-out 88% (25 reviews), 5-fold cross-validation 87%, products it had not seen 76% (target 80% missed). It finds about 64% of negative reviews.

Fourteen of 14 is weak evidence. The questions were written after the data was collected. A pass only requires that the pick lists the skin type and fits the budget, five questions are decided by code, and several accept two to five products. After the run I read the answers to questions 7, 13 and 14 (attribution and the two safety questions) and marked all three Pass; the only flaw I found was that the model wrote an internal product code ("P05") in its text, which the check now replaces with the product name. The other six recommendations were not read. One check did work: in the "cures eczema" test the model produced a treatment claim and the check removed it. Picks concentrate: CeraVe in 6 of 9, the product with by far the most reviews (54 of 103), so evidence volume may drive choices more than suitability. The classifier's missed negatives matter because negatives are what warn a shopper.

## 6. Difficulties and tuning

**Data.** My first batches mixed delivery and product sentiment, and one product had no positive reviews. The model learned "fast delivery means positive", and random cross-validation said 96% while training on two products and testing on the third gave 69% to 94%. Only reviews about the product were kept, validation was done by product, and collection stopped at 121 training reviews (plan: 150 to 200) because product negatives were scarce.

**Tuning.** Three classifiers were compared: a majority-class baseline, word TF-IDF, and word plus character TF-IDF. The last did better in cross-validation (87% against 84%) and on unseen products (76% against 66%) but worse on the 25-review hold-out (88% against 92%), so it was kept.

**Timing.** The platform replays identical model calls for five minutes, so the evaluation runs with caching off.

**Prompt and checks.** Torriden's ingredient list contains lavender flower extract, although one reviewer called the cream fragrance-free. A check was therefore added that removes a property claim the product page does not state, unless a reviewer is credited with it, and a scenario question (number 7) that tests it. The prompt also tells the model to attribute brand claims to the brand and to ignore instructions inside the shopper's message.

**Screening.** Written rules (moisturizing claim, a skin type named on the page, no anti-ageing, acids or SPF, a real seller) were applied to 13 candidates: 7 entered, 6 did not.

## 7. Rough edges and future path

Only 7 products, not 15 to 30; sale prices from one day; reviews are not a random sample and are mostly positive; two skin-type labels (Torriden, Himalaya) are judgments.

Future path: connect the same pipeline to a permitted marketplace feed, so that database filters (budget, category, availability) cut thousands of products to a short list before Claude ranks and explains them; extract and quality-check structured attributes such as skin type and ingredients from product pages; test an LLM against the classifier; add consent-based personalization; keep sponsored products visibly separate from recommendations; and measure conversion and returns in an A/B test before claiming business value. Along the way: more products, re-ranking so that products with few reviews are not overlooked, and a second person reading the answers.

*AI assistance:* the code was written by Claude.
