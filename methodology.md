# How ElectionTracker works

A public-data research forecast for the 2026 U.S. House, Senate and governor elections. This page describes the complete method behind the current single-correction model, including its ordinary race model, candidate exceptions, election rules, correlated simulations and historical evaluation.

**Research status.** The model is a combination of regularized statistical components and predictive simulations. It is not a fully Bayesian joint posterior or a completed dynamic model of voter opinion. The current rebuild failed its frozen full consistency screen at every evaluated horizon. Its adoption is a research choice; numerical audits do not establish superior forecasting accuracy.

The broad sequence is: verify the contest and information available at the cutoff; build geographic fundamentals and a polling average; estimate a race margin or a separate candidate event; apply the appropriate final-election rule; and simulate every contest together. This preserves the possibility that many forecasts are wrong in the same direction.

## Notation and forecast targets

| Symbol | Meaning |
| --- | --- |
| $i,o,s,c,h$ | Race, office, state, election cycle and days before election |
| $y_i$ | Observed Democratic-minus-Republican margin, in percentage points |
| $F_i$ | Fundamentals plus the shared office polling signal |
| $P_i$ | Raw aggregated local D-minus-R polling margin |
| $\mu_i,\sigma_i$ | Final predictive margin center and standard deviation |
| $p_i,q_i$ | Underlying D/R win probability and rule-adjusted final-win probability |
| $B$ | Number of aligned simulations: 20,000 |

The registry contains 435 voting House seats, 35 Senate elections and 36 state governor elections. Territorial governors and nonvoting delegates are outside this universe. A listed contest can lack an unconditional winner estimate.

Historical margins use $100(D-R)/V$, where $V$ is the reported total across candidates, rather than renormalizing the outcome to D plus R. Poll margins are reported D% minus R%; undecideds are not automatically redistributed. Candidate-level polling uses a separate normalization described below. An independent winning and that winner joining the Democratic caucus are different events. Missing information remains missing rather than becoming a zero probability.

## 1. Information available at the cutoff

Poll questions must match the intended contest and candidate identities. Fieldwork, claimed publication dates and first-observation or revision timestamps are checked before aggregation. Deduplication prevents the same imported question, overlapping releases or electorate subgroups from being treated as independent surveys. Related firms or respondent panels that cannot be identified can still create dependence.

Current polling comes from public VoteHub data and a separately reviewed New York Times supplement. Historical polling comes from archived FiveThirtyEight files; older pollster-rating archives supplement earlier years with weaker timing evidence. Official FEC and House Clerk records supply federal outcomes, with separate verified final-winner repairs for runoffs and ranked-choice events. Governor histories and candidate compilations retain their source tiers and coverage limits.

A prior election occurring before a forecast is necessary for a valid feature, but does not prove that a retrospectively downloaded spreadsheet revision was available then. Original publication and revision histories remain incompletely certified. The backtests are chronological retrospective evaluations, not certified reconstructions of everything publicly knowable on each historical day.

## 2. Geographic fundamentals

Fit separate office models at each daily forecast horizon. The feature vector is

$$
x_i=\left(1,\frac{P_i^{\rm pres}}{20},\frac{L_i}{20},
\mathbf1[L_i\text{ missing}],H_c,\mathbf1[\text{geography fallback}]\right).
$$

$P_i^{\rm pres}$ is an earlier presidential margin on the applicable state or congressional map. The 2026 forecast uses 2024 presidential voting. House geography uses map-specific reaggregations; matching district numbers do not establish comparable boundaries. House omits the lagged office-result feature. Regular Senate races use the comparable earlier seat/class result; specials lack that regular-seat prior. Governors use the most recent eligible comparable office election. A missing $L_i$ is entered as zero alongside its explicit missingness indicator.

$H_c$ is $+1$ in a midterm under a Democratic president, $-1$ under a Republican president, and zero in presidential-election years. The ordinary fundamentals do not include explicit current-candidate incumbency, fundraising, approval, GDP or demographic coefficients.

Let $N$ be the number of training rows, $C$ the number of earlier cycles and $n_c$ their cycle counts. The ridge estimate is

$$
\widehat\beta_o=(X^TWX+\Lambda)^{-1}X^TWy,
\quad W_{ii}=\frac{N}{C n_c},
\quad \Lambda=\operatorname{diag}(0,1,1,1,1,1).
$$

Every election receives equal total weight. The intercept is unpenalized; the other coefficients have fixed ridge penalty one. At least three earlier cycles and 25 eligible races are required. Historical target-election outcomes never train their own fundamentals.

## 3. Ordinary D/R polling average

The ordinary polling window is 45 days. A record without verified publication timing receives at least a three-day lag after fieldwork ends. Among related questions from one survey, likely-voter screens take precedence over registered-voter screens and other populations. Exact duplicate answers are collapsed. Connected overlapping fieldwork from the same case-folded pollster becomes one survey family; retained margins are averaged and the family sample size is their maximum.

For family $j$ with age $a_j$ days and sample size $n_j$, the initial weight is

$$
v_j=\sqrt{\min(n_j,1200)}\,2^{-a_j/14}.
$$

Missing or nonpositive sample size receives unit sample weight before time decay. A firm's total contribution is capped at the weight of its strongest family:

$$
w_j=v_j\frac{\max_{k\in f(j)}v_k}{\sum_{k\in f(j)}v_k},
\qquad P_i=\frac{\sum_jw_jm_j}{\sum_jw_j}.
$$

The square-root sample weighting, 14-day half-life and pollster cap are structural choices. They are not estimated pollster-quality ratings or firm-specific partisan effects. A large coverage count does not imply the same number of independent information units. The ordinary average leaves undecideds in the reported denominator.

## 4. Shared office polling environment

Available race polls also provide an office-wide signal:

$$
g_{oc}=\operatorname{median}_{j\in\mathcal P_{oc}}
(P_j-x_j^T\widehat\beta_o),
\qquad F_i=x_i^T\widehat\beta_o+\alpha_o g_{oc}.
$$

The signal is zero when fewer than three polled rows are available. The response $\alpha_o$ is a zero-intercept least-squares slope between earlier-cycle signals and the office's mean fundamentals errors, clipped to $[0,1]$. It is learned from race polling, not a generic-ballot series. Consequently, $F_i$ and local polling $P_i$ contain correlated information; the blend does not treat them as independent measurements.

## 5. One fitted correction for eligible polled races

For source-valid ordinary polled D/R plurality races, estimate a convex blend and one intercept jointly:

$$
\mu_i=F_i+w_o(P_i-F_i)+a_o,\qquad 0\leq w_o\leq1,
$$

$$
(\widehat w_o,\widehat a_o)=\arg\min_{w,a}
\left[\sum_{c<t}\frac1{n_{oc}}\sum_{i\in(o,c)}
\{y_i-F_i-w(P_i-F_i)-a\}^2+4a^2\right].
$$

Each earlier cycle has total loss weight one. The intercept penalty represents four cycle-equivalents; there is no penalty on $w$. The exact quadratic solution is compared with solutions at $w=0$ and $w=1$. Support requires two earlier cycles, 30 eligible races and variation in $P-F$. This is regularization, not a claim that the complete forecast is a Bayesian posterior.

An illustrative frozen example is the **October 27, 2026 seven-day profile**. These are fitted mean parameters for eligible races, not universal coefficients or a choice among dates. Other forecast days use their own prepared fits.

| Office | Local polling weight $w_o$ | Intercept $a_o$, percentage points |
| --- | ---: | ---: |
| House | 0.685465280 | -1.248894219 |
| Senate | 0.824040780 | -1.260003638 |
| Governor | 0.926787484 | -1.761816252 |

Neither legacy additive correction is stacked onto an active single-correction mean. The current Maine Senate race receives a separately authorized transfer of the Senate blend before its ranked-choice link. This exception is restricted to that reviewed 2026 event; it is not a general override for unsupported nonplurality races.

Protected, unpolled and unsupported means retain the explicit native fallback:

$$
\mu_i^{\rm native}=(1-w_N)F_i+w_N(P_i+b_o)+d_o.
$$

Here $b_o$ estimates earlier actual-minus-poll errors and $d_o$ earlier prequential combined-forecast errors. Unpolled races set $w_N=0$. Those offsets estimate different quantities; the existence of two terms alone does not prove double counting. Separate candidate and rule branches can supersede the latent D/R target entirely.

## 6. Complete historical errors and covariance

Construct final-policy residuals $e_i=y_i-\mu_i^{\rm final}$ recursively, using only models available before each target election. Include valid unpolled and native-fallback predictions. Missing margins and unsupported candidate-share targets do not become ordinary margin errors. Complete final-error history begins in 2016; no 2012/2014 complete errors are invented.

For supported races, the rebuilt uncertainty has exactly zero additional location. Residual means are diagnostics, never another offset. Its covariance is

$$
\Sigma_{ij}=v_N+\mathbf1[o_i=o_j]v_{O,o_i}
+\mathbf1[s_i=s_j]v_S+\mathbf1[i=j]v_{I,o_i,z_i},\qquad v\geq0.
$$

The mutually exclusive strata $z$ are geography fallback first, then polled, then unpolled. National and state loadings have equal percentage-point magnitude across supported offices. Nonnegative variance components make this covariance positive semidefinite. This is a pooled shared-state factor, not a free 50-by-50 correlation matrix. Latent margin correlation is $\Sigma_{ij}/\sqrt{\Sigma_{ii}\Sigma_{jj}}$; winner-event correlation also depends on each race's probability and event rule.

Diagonal moments average squared residuals within election/office/stratum. Pair moments average distinct-race products within state or state-pair units, then average units equally. Each election has equal objective weight, divided equally across its available moment categories. Many House districts therefore do not become many independent election-level observations.

For office $o$, let $u_o$ be its equal-cycle pooled uncentered MSE. Normalize moment $m_j$ by $\eta_j=\sqrt{u_{o_1}u_{o_2}}$. Let $B_{jk}$ identify which variance components enter moment $j$. Parameter units $t_k$ equal the mean supported-office MSE for national/state components, and the relevant office MSE for office/independent components. With $v_k=t_kz_k$, solve

$$
\min_{z\geq0}\sum_jq_j
\left(\sum_k\frac{B_{jk}t_k}{\eta_j}z_k-\frac{m_j}{\eta_j}\right)^2
+\frac1C\lVert z-z_0\rVert^2.
$$

$q_j=1/(C K_c)$ for the $K_c$ available moment categories in election $c$. Shared/office prior coordinates are zero; independent prior variances are their office pooled MSEs. The fixed ridge is one pseudo-election, not a held-out tuning search. A deterministic nonnegative solver checks its optimality conditions. Numerical independent variance has floor $10^{-8}$ percentage-points squared.

Each supported office needs at least two earlier cycles and 30 valid errors. Cross-office bridge support and full-rank shared pair design are required. Otherwise native uncertainty is retained around the supplied final center. The first supported rebuilt covariance evaluation is 2020. No old common-variance floor is imposed on an active rebuilt fit.

## 7. Heavy tails and joint simulations

Let $\sigma_i^2=\Sigma_{ii}$. The ordinary marginal distribution is Student-t with five degrees of freedom, center $\mu_i$ and scale $s_i=\sigma_i\sqrt{3/5}$. Its Democratic D/R probability is

$$
p_i=T_5(\mu_i/s_i).
$$

Symmetric 80% and 95% intervals use $\mu_i\pm t_{5,.9}s_i$ and $\mu_i\pm t_{5,.975}s_i$. For aligned draw $b$,

$$
M_{ib}=\mu_i+\sqrt{3/Q_b}
\left(\sqrt{v_N}Z_{Nb}+\sqrt{v_{O,o_i}}Z_{o_i b}
+\sqrt{v_S}Z_{s_i b}+\sqrt{v_{I,o_i,z_i}}Z_{ib}\right),
\quad Q_b\sim\chi^2_5.
$$

Normal factors are independent before sharing the indicated national, office and state terms. The common tail multiplier is itself an assumption: zero covariance does not guarantee tail independence. There are 20,000 aligned draws, using a fixed reproducible seed.

Binary events allocate exactly $\operatorname{round}(Bp)$ wins to the highest aligned margin ranks. This matches the desired marginal probability to simulation resolution while retaining dependence. It does not draw each race independently. Simulated event frequencies can differ slightly from displayed analytic probabilities because of rounding.

## 8. Final-election rules

Ordinary plurality events use their underlying probability directly. Reviewed nonplurality routes use a separate regularized logistic link to final Democratic winners:

$$
q_i=\operatorname{logit}^{-1}\left[\gamma_0+\gamma_1\operatorname{logit}(p_i)
+\gamma_2I_{\rm runoff}+\gamma_3I_{\rm ranked\ choice}+\gamma_4I_{\rm legislature}\right].
$$

Clip input probabilities to $[10^{-6},1-10^{-6}]$. Fit weighted logistic negative log likelihood plus $4\lVert\gamma-(0,1,0,0,0)\rVert^2$. If $R$ is the number of unique training race/cycle pairs, $C$ the cycle count, $R_c$ that cycle's unique count and $r_{ic}$ its repeated horizon count, record weight is $R/(C R_c r_{ic})$. Snapshots at 1/7/14/15/30 days share race weight; the target cycle is excluded at every horizon.

Calibrated wins use the existing margin ranks at $q_i$. The link predicts final winners; it does not estimate voters' second choices, exhausted ballots or runoff turnout. Directly comparable supported historical Maine Senate margin validation is unavailable. Earlier Maine-only rebuilt-link ranked-choice scores worsened at every tested date; that adverse evidence remains part of the record.

## 9. Named candidate fields

Independent or other nonordinary fields use candidate polling when the reviewed route supports it. This branch has a **60-day window and 21-day half-life**, rather than the ordinary polling settings. It selects the most complete observed field, uses frequency to break completeness ties, keeps one preferred question per survey and nonoverlapping pollster releases, and averages normalized field shares with age-only weights. Smaller head-to-head questions never fill omitted candidates with zero.

Candidate-share errors are learned from earlier cycles, balancing elections. Roles are Democratic, Republican, major-other and minor-other; major-other means at least 10% in polling. Separate role estimates require 25 observations across two cycles. Sparse serious-independent errors use zero bias and variance at least as large as pooled major-candidate variance or observed independent MSE. The 10% error-role threshold differs from the 5% threshold used for material nonmajor event routing.

For $n$ candidates, define $C_n=I-\mathbf1\mathbf1^T/n$. Role standard deviations generate the centered covariance

$$
K=C_n\operatorname{diag}\left(\sigma_j^2\frac n{n-1}\right)C_n.
$$

Start Democratic/Republican loadings at $+1/2,-1/2$, other loadings at zero, and center the loading vector $l$ across the field. For shared-margin standard deviation $s$, the maximum admissible coupling is

$$
\lambda=\min\left(1,\sqrt{\frac{1-10^{-12}}{s^2l^TK^+l}}\right),
\qquad K_{\rm residual}=K-(\lambda s)^2ll^T,
$$

with $\lambda=1$ when the denominator is zero. $K^+$ is the pseudoinverse. Candidate residuals use $K_{\rm residual}$ and their own t5 tail, then add the aligned common shock with loading $\lambda l$. Before constraining shares, candidate $j$ has score $\widetilde s_j=\overline p_j+b_{r(j)}+\varepsilon_j$: normalized polling share plus its fitted role bias plus the composed error. Recenter scores to total 100, clamp negative shares and renormalize; the plurality winner is their largest share. Because independent and shared terms can have different tail multipliers, this composed field is not one exact multivariate t distribution. The cap preserves the calibrated centered covariance instead of widening it to accommodate common error.

Historical candidate-error panels exclude fields that omit observed top finishers or another substantial candidate. This retrospective outcome-based coverage check is a limitation of the error-training evidence, not proof that the complete historical field was known at the forecast cutoff.

Results remain conditional until ballot completeness and write-ins are reviewed. Explicit reallocated finalist polls provide conditional ranked-choice pair scenarios; they do not supply qualification probabilities. Alaska House therefore retains no unconditional winner point estimate. Mechanical ranked-choice/runoff routines do not invent transfer preferences. Only complete reviewed same-party fields support party certainty.

For evaluated nonplurality fields supported only by first-choice polling, a conservative bounds branch resolves a simulated draw only when a candidate has strictly more than 50% of that field’s shares. Otherwise every possible final winner remains admissible. Democratic-party lower and upper indicators are $I(\text{majority winner is D})$ and $1-I(\text{majority winner is non-D})$. Caucus bounds use documented affiliations, leaving an undeclared independent unresolved. This branch supplies bounds rather than an unconditional point estimate; it does not replace the live source-route review.

## 10. Fields with only one major party

A separate logistic model predicts whether the present printed major party wins against all alternatives. Its features are

$$
x_i^{(1)}=(1,\epsilon_i P_i^{\rm pres}/20,I_{\rm major\ incumbent},
I_{\rm other\ incumbent},I_{\rm other\ listed}),
$$

where $\epsilon_i=+1$ for Democrats and $-1$ for Republicans. Fit equal-election-weighted logistic negative log likelihood plus $\tfrac12\beta^T\operatorname{diag}(.1,2,2,2,2)\beta$. The inverse fitted Hessian supplies a Laplace coefficient covariance. Average logistic probabilities over 2,048 coefficient draws; coefficient intervals exclude specification and ballot-vintage uncertainty.

Wins use aligned margin ranks oriented toward the present major party. This predicts that party versus “other,” not individual alternative candidates. Unknown alternative caucuses retain bounds unless a documented affiliation resolves them. Rare independent victories and reconstructed historical fields limit calibration despite strong aggregate safe-seat scores.

## 11. House and Senate control

Each draw sums fixed Democratic caucus seats and simulated lower/upper caucus indicators:

$$
S_b^-=S_{\rm fixed,D}+\sum_iD_{ib}^-,\qquad
S_b^+=S_{\rm fixed,D}+S_{\rm fixed,unknown}+\sum_iD_{ib}^+.
$$

An unresolved contest contributes zero to the lower total and one to the upper. Unknown independent affiliation can similarly separate the two arrays. For threshold $T$, control bounds are $B^{-1}\sum_b I[S_b^-\geq T]$ and $B^{-1}\sum_b I[S_b^+\geq T]$. House uses 218; Senate uses 51. A 50-seat Senate outcome is separate from this strict-majority target. Governors have no national-control event.

Expected-seat bounds and histograms use these same aligned arrays. Scenario bounds describe unresolved event/caucus assumptions; they are not confidence intervals. House map colors describe expected Democratic seat share within each state, not a statewide House-win probability or district boundaries. Senate states with two contests show an average probability; the race list supplies individual estimates.

### Headline reference scenario

The single House and Senate headline percentages are **assumption-based reference scenarios**, not averages of the lower and upper bounds. They include established Democratic-caucusing independents. A winner without a modeled Democratic-caucus commitment is left unassigned and does not count toward Democratic control; this does not assign that winner to Republicans.

For the reference scenario only, FL-13, MT-01, NV-02 and TX-28 use their existing latent Democratic-versus-Republican distributions, conditional on no nonmajor winner. Their individual unconditional full-field odds remain withheld. Alaska Senate assumes Mary Peltola and incumbent Dan S. Sullivan reach the final round. Alaska House assumes Nick Begich and Bill Hill reach the final round **and assumes Hill organizes with Democrats if he wins**. No caucus commitment is used here; Hill→D is an explicit reference assumption, not a verified affiliation. A [direct June interview](https://www.alaskasnewssource.com/2026/06/17/guess-well-find-out-hill-wont-say-if-hell-caucus-with-democrats-challenge-against-rep-nick-begich/) records his refusal to commit. Finalist qualification probabilities are not modeled.

The Alaska scenarios use reviewed final-round questions, not first-choice shares or forced exhausted-ballot transfers. Data for Progress October 8–16 Q18 reports Peltola 51–Sullivan 49 among 671 continuing ballots; Q23 reports Begich 51–Hill 49 among 659. Total survey size is 694. The separately verified AARP final-round questions and the previously reviewed Bullfinch House question remain included while within the unchanged 60-day candidate-poll window. The current official Senate field lists Daniel J. Sullivan Jr. as undeclared; he is a different person from incumbent Dan S. Sullivan. Those omitted candidates remain a qualification risk.

For each of the same 20,000 joint draws, the reference scenario sums its Democratic-aligned winner indicators. The displayed percentage is the fraction reaching 218 House seats or 51 Senate seats. A 50-seat Democratic Senate caucus does not meet that threshold with a Republican vice president. The reference calculation preserves the repaired ensemble’s shared shocks and paired structural bounds; no independent coin flips or new historical fits are introduced.

Sources: [Data for Progress original questionnaire](https://www.filesforprogress.org/datasets/2026/10/dfp_alaska_oct_2026.pdf), [AARP questionnaire](https://www.aarp.org/content/dam/aarp/research/topics/voter-opinion-research/politics/2026-midterm-election-poll-alaska-annotated-questionnaire.doi.10.26419-2fres.01065.017.pdf), [official Alaska general-election field](https://www.elections.alaska.gov/candidates/?election=26GENR), and [official election rules](https://www.elections.alaska.gov/election-information/).

This complete reference scenario has not been historically or prospectively validated. The bounds and all assumptions remain available beneath each headline. Reference percentages cannot receive an exact model-minus-market edge. Earlier dated releases remain archived, and current history is separated by both fitted-policy and event-review identity.

## 12. Historical evaluation and known weaknesses

The complete comparison evaluates 2018, 2020, 2022 and 2024 with earlier-cycle fitting. Fourteen days is primary; 1/7/30 days are sensitivities. Three fixed seeds each use 20,000 draws. Four arms separate original native, predecessor live, single-correction means with native errors/links, and the full rebuild. Identical outcome-free guards apply after exact original-native archive reproduction. Missing truth is unscored; conditional and bounded events remain identified.

For observed winner $y\in\{0,1\}$,

$$
\operatorname{Brier}=(p-y)^2,\qquad
\operatorname{LogLoss}=-y\log p-(1-y)\log(1-p).
$$

Margin scores include MAE, RMSE, coverage and weighted interval score. With $(x)_+=\max(x,0)$,

$$
IS_\alpha=(u-l)+\frac2\alpha(l-y)_++\frac2\alpha(y-u)_+,
\qquad WIS=\frac{.5|y-\mu|+.1IS_{.2}+.025IS_{.05}}{2.5}.
$$

Discrete seat CRPS is $\sum_{k=0}^{S-1}[F(k)-I(y\leq k)]^2$. For incomplete seat distributions, pointwise CDF bounds yield conservative score bounds; those differ from scores of the lower/upper endpoint scenarios.

| Primary 14-day Brier | Predecessor | Complete rebuild |
| --- | ---: | ---: |
| House | 0.03783 | 0.03877 |
| Senate | 0.04641 | 0.04923 |
| Governor | 0.06527 | 0.05691 |

Lower is better. Primary governor averages improve; House and Senate worsen against predecessor live. Safe races dominate pooled scores. Competitive comparisons fix membership from native 10–90% probabilities; equal-election summaries, office/cycle breakdowns and omission checks are retained separately. Every full consistency screen fails: 355 checks at 14 days, 366 at 1 day, 272 at 7 days and 360 at 30 days. These are dependent metric/comparator checks, not independent hypothesis tests. Dates and seeds reuse four election blocks. Previously inspected historical elections and incomplete publication vintages do not establish prospective 2026 accuracy or precise chamber calibration.

## Reproducibility, independence and sources

All exact daily profiles from 1 through 30 days are prepared before release. Runtime selects the calendar-day profile without refitting, interpolation or choosing a date based on held-out scores. Source, model and review hashes must match; changed or missing bindings stop release. Policy histories are separated so earlier model versions are not silently joined to the current trend. Original data, failed studies and prior forecasts are preserved.

Hosting serves a dated static snapshot; collection, calculation and release checks run locally. A successful update requires a valid local build and publication, rather than the host fitting the model itself. If an update cannot pass release or publication checks, the last valid published snapshot retains its original dates. On and after November 3, the pre-election forecast freezes to a matching reviewed pre-election release. Results and subsequent runoff updates require separate modeling and review.

Market odds are comparison data only. Bid/ask spreads, stale quotes and settlement differences remain visible; prices never enter this forecast. Generic-ballot series, demographics, economic indicators, early-vote reports and neural challengers are not current forecast inputs. Party registration among early voters is not vote choice. Read-only price comparisons use [Kalshi](https://docs.kalshi.com/getting_started/quick_start_market_data) and [Polymarket](https://docs.polymarket.com/market-data/prices-order-books) data, with exact event matching required.

Primary and original-author sources include [VoteHub polling](https://votehub.com/polls/api/) ([CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)), [New York Times polling](https://www.nytimes.com/interactive/polls/latest-polls.html), [FiveThirtyEight author repository](https://github.com/fivethirtyeight/data), [FEC election results](https://www.fec.gov/introduction-campaign-finance/election-results-and-voting-information/), [House Clerk statistics](https://clerk.house.gov/member_info/electionInfo/2024/statistics2024.pdf), [The Downballot map calculations](https://www.the-downballot.com/p/the-downballots-calculations-of-presidential), and [The Green Papers candidate compilation](https://www.thegreenpapers.com/G26/), supplemented by official state records. A compilation is not equivalent to a certified complete ballot.

Explorer state outlines use U.S. Census Bureau geometry via [us-atlas 3.0.1](https://cdn.jsdelivr.net/npm/us-atlas@3.0.1/states-albers-10m.json), geometry year 2017, with Alaska and Hawaii inset, under the [ISC license](https://github.com/topojson/us-atlas/blob/master/LICENSE). These state outlines are presentation geometry, not the district maps used for House presidential features.

### us-atlas ISC notice

Copyright 2013-2019 Michael Bostock

Permission to use, copy, modify, and/or distribute this software for any purpose
with or without fee is hereby granted, provided that the above copyright notice
and this permission notice appear in all copies.

THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES WITH
REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF MERCHANTABILITY AND
FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR ANY SPECIAL, DIRECT,
INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES WHATSOEVER RESULTING FROM LOSS
OF USE, DATA OR PROFITS, WHETHER IN AN ACTION OF CONTRACT, NEGLIGENCE OR OTHER
TORTIOUS ACTION, ARISING OUT OF OR IN CONNECTION WITH THE USE OR PERFORMANCE OF
THIS SOFTWARE.

Source: https://github.com/topojson/us-atlas/blob/master/LICENSE

### Current method identity

Method version: `national-consistent-single-correction-ensemble-v1`.

Policy SHA-256: `caa2f6fd9e8f033e52297b95a64a1da3e049687487ac424cf603b138a13335a0`.

Completed rebuild study: `7613748b7e76e4bc1db8c8201de40b0c53e3628ed9c34c4bbe167ee20419a7ff`.

The fitted model retains its October 27, 2026 research identity. The October 29 event review repairs Alaska finalist evidence and adds an explicitly assumed chamber reference scenario; it does not refit the model or upgrade the failed statistical screen.

