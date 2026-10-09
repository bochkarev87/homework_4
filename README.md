# homework_4
HW4. Association Rules on the UCI Online Retail Dataset
Repository: Live page: [insert link to the GitHub Pages site]
Abstract
We mine association rules from 17,080 cleaned invoices of the UCI Online Retail dataset [1] using a
level-wise Apriori miner [2] implemented in JavaScript, and evaluate rules with support, confidence and lift,
which compares co-occurrence with independence [3]. With minimum support 1% (171 baskets) and
minimum confidence 30% the miner returns 950 rules, all with lift above 1. We present one rule worth
investigating (LUNCH BAG RED RETROSPOT -> LUNCH BAG PINK POLKADOT, lift 7.46) and one rule
to reject (JUMBO BAG ALPHABET -> WHITE HANGING HEART T-LIGHT HOLDER, lift 1.05), study the
effect of thresholds, and propose a validation experiment.
1. Implementation
Each InvoiceNo is one basket; items are identified by StockCode and labelled with Description. The
provided data file already contains the cleaned dataset (384,911 rows, 17,080 baskets, 3,653 items;
cancellations, non-products and guest checkouts removed). The seven required functions in script.js
were implemented as follows.
- dedupeBasket / countItemset: a basket is treated as a set (first-appearance order kept); counting
intersects the posting lists of an inverted index, starting from the smallest list, and returns 0 for an empty
or unknown itemset.
- computeSupport / Confidence / Lift: support = count(A,B)/N, confidence = count(A,B)/count(A), lift =
confidence / (count(B)/N); each returns {value, defined} and is undefined (not Infinity or NaN) on a zero
denominator.
- findFrequentItemsets: Apriori. Level k+1 candidates are produced by joining k-itemsets that share a
prefix and are pruned if any k-subset is infrequent (downward closure); candidates are counted by
intersecting the sorted basket-id lists of their parents, so no candidate triggers a full scan.
- generateRules: every non-empty proper subset of a frequent itemset is an antecedent and its
complement the consequent, so both A -> B and B -> A (and many-to-one splits) are produced; rules
below the confidence threshold are dropped. Rules are sorted by lift. The lift > 1 filter is applied in the
analysis, as the task states; at the thresholds used here it removes nothing (see 2.3).
2. Business and algorithmic analysis
2.1 Useful rule versus misleading rule
Rule (A -> B) count(A
,B)
count(
A)
count(
B)
suppo
rt
confiden
ce
lift
Investigate: LUNCH BAG RED
RETROSPOT -> LUNCH BAG PINK
POLKADOT
523 1,288 930 3.06% 40.61% 7.46
Reject: JUMBO BAG ALPHABET -> WHITE
HANGING HEART T-LIGHT HOLDER
90 750 1,959 0.53% 12.00% 1.05
Worth investigating. The antecedent appears in 1,288 of 17,080 baskets (7.5%), so the rule is
actionable. Of those baskets 40.6% also contain the pink polkadot bag, although that bag appears in only
5.4% of all baskets (930/17,080); lift = 0.4061 / 0.0545 = 7.46, i.e. the co-purchase is about seven times
more frequent than independence predicts. High confidence alone would not show this (the baseline must
be known); lift does. Action: a 'complete the set' recommendation slot, or a mixed lunch-bag multipack, on
the red retrospot product page.
Reject. The rule passes the loosest thresholds (support 0.5%, confidence 10%) and shows 12.0%
confidence, but the consequent is the most frequent item (1,959 baskets, 11.47%). Lift = 0.120 / 0.1147 =
1.05: buying the alphabet bag barely changes the chance of buying the T-light holder, so the rule is mostly
a reflection of the consequent's popularity. A naive reading ('12% of alphabet-bag buyers also buy hearts,
promote it') overlooks that 11.5% of all customers do. Another observation: no item occurs in more than
11.47% of baskets, therefore every rule with confidence at least 30% has lift at least 2.6, and vacuous
rules can appear only at low confidence thresholds. A stronger-looking but still low-value example is HERB
MARKER THYME -> HERB MARKER ROSEMARY (confidence 94.6%, lift 85.1): the count(A) is only 186
baskets (1.1%) and the items are members of one product set, so the rule mostly describes customers
completing a set they already intend to buy.
2.2 Rule direction
For the rule above, confidence(RED -> PINK) = 523/1,288 = 40.6%, while confidence(PINK -> RED) =
523/930 = 56.2%. The numerator is the same; only the denominator (the size of the antecedent group)
differs, so confidence depends on direction. Lift is 7.46 in both directions because it is symmetric.
2.3 Threshold trade-off
Final thresholds: minimum support 1% (171 baskets) and minimum confidence 30%. With N =
17,080 and about 22.5 distinct items per basket, a pattern in fewer than 171 baskets is too rare to promote;
30% is well above the largest item baseline (11.47%), so every kept rule has lift of at least 2.64 (all 950
rules have lift above 1). This setting yields 950 rules, a list a merchandiser can read.
Min support
(baskets)
Frequent itemsets (size 1 / 2 / 3 / 4+) Rules, conf
30%
Rules, conf
60%
Rules with lift > 1
(conf 30%)
0.5% (86) 5,255 (1,320 / 2,206 / 1,161 / 568) 10,723 3,958 10,723
1% (171) 1,219 (691 / 413 / 109 / 6) 950 238 950
2% (342) 294 (242 / 51 / 1 / 0) 96 20 96
3% (513) 115 (109 / 6 / 0 / 0) 12 5 12
Raising minimum support removes itemsets and therefore rules: by downward closure every subset of a
frequent itemset is frequent, so a higher threshold prunes whole branches of the itemset lattice, and long
itemsets vanish first (no itemset of size 3 or more survives at 3%). Raising minimum confidence does not
change the itemsets, only the splits kept (10,723 to 3,958 at 0.5% support). Low thresholds give a long,
noisy list (19,058 rules at 0.5% / 10%, of which 104 have lift below 1.5 and 2 have lift at most 1); high
thresholds give a short, reliable but conservative list that misses niche but real patterns.
2.4 Cross-sell use case and its limitation
Feature: on a LUNCH BAG RED RETROSPOT page show 'Customers also buy LUNCH BAG PINK
POLKADOT' and offer a mixed multipack. Main limitation: a rule is a frequency statement about the
observed period (1 Dec 2010 to 9 Dec 2011 [1]) only, and the retailer sells to many wholesalers [1]; large
trade baskets contain many colour variants of the same bag, so the rule may describe wholesale
range-buying more than the choice of a retail customer.
2.5 Association is not causation
A high lift shows that two items co-occur, not that promoting A makes customers buy B. For the lunch-bag
rule a plausible confounder is the buyer type and occasion: wholesalers and party or school-term shoppers
buy several designs of the same bag regardless of any promotion, so both items rise together because of
a hidden third factor (customer type or season). Promoting A to a retail customer may not increase B at all,
and customers who would have bought B anyway inflate the observed confidence.
2.6 Validation plan (requires data the starter does not contain)
The exported baskets have no timestamps, so no temporal check is possible with the starter, and none is
simulated. The plan therefore needs new data. (a) Held-out period: recompute count(A,B), count(A),
count(B), confidence and lift on a later period of invoices (for example the following quarter, using the
original InvoiceDate). Accept the rule as stable if confidence stays at least 30% and lift at least 1.5 with an
antecedent count above 171; otherwise reject. (b) A/B test: randomly show the recommendation slot to a
treatment group and not to a control group on the red retrospot page, and compare the attach rate of the
pink polkadot bag. To detect an uplift of 3 percentage points over a 40% control rate (alpha = 0.05, power
0.8) about 4,200 customers per arm are needed (two-proportion formula); because only 1,288 baskets in
this dataset contain the antecedent, the test would have to run across all lunch-bag variants or for several
months. Decision rule: ship if the lower bound of the 95% confidence interval of the incremental attach rate
is above 0 and the extra margin covers the cost of the slot; otherwise stop.
3. Verification and AI usage
The implementation and this report were produced with the help of an AI assistant (Claude). Checks that
were performed: (1) the built-in self-checks of the starter, 11/11 PASS, executed in Node.js with a minimal
DOM stub for the one rendering check; (2) an independent brute-force recount, scanning all 17,080
baskets directly, of count(A,B), count(A) and count(B) for all 950 rules at 1% / 30%, with 0 mismatches; (3)
a brute-force count of all item pairs, which found the same 413 frequent pairs with identical counts; (4) the
arithmetic in Section 2.1 and 2.2 recomputed from the displayed counts. Not verified: the source workbook
itself was not re-downloaded, and no causal claim was tested.
References
[1] D. Chen, S. L. Sain and K. Guo, "Data mining for the online retail industry: A case study of RFM model-based
customer segmentation using data mining," J. Database Marketing & Customer Strategy Management, vol. 19,
no. 3, pp. 197-208, 2012, doi: 10.1057/dbm.2012.17. Dataset: UCI Machine Learning Repository, Online Retail, id
352, https://archive.ics.uci.edu/dataset/352/online+retail
[2] R. Agrawal and R. Srikant, "Fast algorithms for mining association rules," in Proc. 20th Int. Conf. Very Large Data
Bases (VLDB), Santiago, Chile, 1994, pp. 487-499.
[3] S. Brin, R. Motwani and C. Silverstein, "Beyond market baskets: Generalizing association rules to correlations," in
Proc. ACM SIGMOD Int. Conf. Management of Data, 1997, pp. 265-276.
