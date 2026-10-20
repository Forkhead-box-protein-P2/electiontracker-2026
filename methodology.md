# ElectionTracker 2026 methodology

This is a public-data research baseline for the 2026 U.S. House, Senate and governor elections. It is a retrospective, historically evaluated model with incomplete historical publication vintages and few independent chamber observations. It is not a certified real-time historical replay.

## Ordinary races

Separate regularized regressions for House, Senate and governor vote margins use prior presidential results, comparable prior statewide office results when available, missing-data indicators, and the president's party in a midterm. House prior-office margins are omitted because district numbers alone do not establish comparable geography. An office-level polling signal measures the median departure of current race polls from fundamentals.

Ordinary polls use a 45-day window, 14-day recency half-life, capped sample-size weights, survey and overlapping-fieldwork deduplication, and capped influence per named firm. The model does not currently estimate individual pollster quality grades or partisan house effects. A polling-bias correction and convex polling/fundamentals blend are learned from earlier elections. A second correction estimates remaining bias in earlier out-of-sample blended forecasts; it is pooled by office, not specific to an individual state.

Historical forecast residuals estimate uncertainty with national, office, state and race components. A fixed five-degree Student-t distribution allows heavy tails. Ordinary win probabilities are calculated from the resulting margin distribution; 20,000 correlated simulations produce chamber seat distributions. Ordinary polled races within an office currently share a polling weight and error stratum at a given horizon, rather than adapting to each race's polling depth and disagreement. The national generic ballot does not directly enter the live forecast.

## Events and chamber accounting

Candidate-field and election-rule components handle reviewed exceptions. Conditional candidate fields, two-major-party approximations, unresolved finalist qualification and undeclared independent caucuses stay explicit. A missing minor-party probability is not a measured zero. Chamber ranges are bounds across unresolved scenarios, not confidence intervals. House majority requires 218 seats; the Senate target shown here is at least 51 Democratic-caucus seats, with exactly 50 treated separately. Market settlement can use a different definition.

## Historical evaluation

Each scored cycle uses fits from earlier elections. Baseline probability evaluations cover 2016–2024; complete combined chamber replays cover 2020, 2022 and 2024. Separate older-history research reaches 2012. We assess Brier score, log loss, margin error, interval coverage, competitive races and seat distributions. Retrospective source revisions and repeated examination of the same elections limit these results. Safe races can dominate pooled scores.

## Inputs and updates

Market prices remain comparison-only. Numerical differences require a fresh quote and matching reviewed event; proxy contracts retain their labels without a numerical edge. Pollster-effect, demographic, early-vote, regional and neural challengers remain separate research unless independently reviewed and adopted. Early-vote registration and county turnout are not candidate votes.

This static explorer shows a dated snapshot generated locally. Hosting does not run the model. Updates depend on successful local collection, release checks and publication, with the Mac awake and online. Failures preserve the last valid site. Forecast and quote dates remain visible; an old snapshot must not be read as live pricing. Pre-election forecasts freeze from November 3; results and runoff updates require separate models.

## Attribution

- VoteHub polling: https://votehub.com/polls/api/ — Creative Commons Attribution 4.0, https://creativecommons.org/licenses/by/4.0/. Transformations include election/candidate filtering, survey deduplication, source review and aggregation.
- The New York Times polling data: https://www.nytimes.com/interactive/polls/latest-polls.html — CC BY 4.0. Changes include general-election filtering, question grouping, candidate matching, overlap exclusions and separately reviewed sample corrections.
- Historical inputs retain citations in the explorer, including the FEC, Office of the Clerk, FiveThirtyEight, the Downballot and the Green Papers.
- Market observations link to the relevant public Polymarket and Kalshi contracts. Displayed prices, retrieval dates and semantic comparison labels are separate from the forecast.
- State boundaries: U.S. Census Bureau via us-atlas 3.0.1, transformed from TopoJSON into SVG paths with Albers USA projection, Alaska/Hawaii insets and coordinate rounding. These are state outlines, not House district boundaries.

The public copy removes machine-local file paths, source links containing opaque access parameters, and unused verbatim market rule text. It preserves forecast values, quote numbers, dates, event scope and source hashes.

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
