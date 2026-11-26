# Changelog

Each qeLib release, newest first. While the major version is 0, a minor
release may change interfaces; a patch release only fixes.

## 0.4.0 - 2026-11-26

### Summary

- **Breaking changes.** A credit index's version after a credit event is
  now described on its own outstanding notional, as the market quotes it.
  Index and tranche objects saved by 0.3.0 after version 1 no longer load.
  `Data.FxRate` takes the date it is observed on. Government z-spreads take
  an optional settlement curve. The flat-spread conversions,
  `Data.BespokeCds`, `Curves.termRateHedges` and index option quote sheets
  change their inputs. Interface changes, below, says what to change.
- **Implied quotes.** CDS and index CDS quotes, swaption, cap, FX and
  index CDS option volatilities, and base correlations, each implied by
  the curves, model or surface that prices them.
- **Credit.** Robust CDS strips correct the quotes they cannot match. New
  quote types hold CDS and index quotes in flat spreads. Index CDS options
  can now use the SABR model. The index and tranche greeks give each
  name's jump to default. Credit events can settle as of the end of the
  valuation date.
- **FX.** FX options are valued at a time of day, to their expiry cut. A
  spot rate can be moved to another date, as a theta needs.
- **Rates.** A rates model's density can be shocked. The SABR calibrations
  and the caplet strip have a Jacobian to the curves.
- **Numerics.** Black and Bachelier implied volatilities and the normal
  distribution use Jäckel's methods.
- **Tools.** The SPI log tools ship with qeLib. Each reported cashflow has
  a type.

### Interface changes

These may need changes to workbooks and scripts written for 0.3.0.

- **A government z-spread is matched by the value today, the settlement
  amount on its own curve.** `Bonds.governmentBondZSpreadPrice` and
  `governmentBondZSpread`, and the bills' `governmentBillZSpreadPrice` and
  `governmentBillZSpread`, take an optional `settlementDiscountCurve`,
  typically OIS, as the government curve fits do. The price at a z-spread
  is now the one at which a trade is worth nothing today: its cashflows
  valued as `pvGovernmentBondZSpread` (`pvGovernmentBillZSpread`) values
  them - on the curve shifted by the spread from its base date, the
  valuation date - equal to its settlement amount discounted on the
  settlement curve, or on the curve itself if that is omitted. Before, the
  cashflows were taken to the settlement date on the curve alone and
  shifted from there, so a holding valued at its z-spread was not worth its
  price, and a security a fit with an OIS settlement curve reprices had a
  small z-spread over it. Now it has none. The spread shifts from the
  valuation date, not settlement, so z-spreads differ slightly even with no
  settlement curve.

- **The flat-spread conversions take and give the spread quote types.**
  `IsdaCds.cdsCurveFromSpreads` and `cdsCurveFromSpreadsJacobian` take a
  `Data.CdsSpreadCurve`, `indexCdsQuotesFromSpreads` and its Jacobian a
  `Data.IndexCdsSpreadQuotes`, and `indexCdoQuotesFromReferenceSpreads`
  and its Jacobian a `Data.IndexCdoSpreadReferenceQuotes`, each with the
  recovery rate, the ISDA discount curve and the holiday calendars, in
  place of the type's fields as arguments. The quotes must be observed on
  the discount curve's base date, the trade date, which the conversions
  used to set. `IsdaCds.cdsCurveToSpreads` gives a `Data.CdsSpreadCurve`,
  not an array, and refuses a curve whose quotes have different coupons
  or that has no quotes. What they give is unchanged.

- **An index version after a credit event is in the market's convention.**
  A version is now described on its own outstanding notional, as the index
  administrator publishes it. What describes a version carries its index
  factor, `indexFactor` (1 by default, and at version 1): `Data.IndexCds`,
  `Data.IndexCdo`, `Data.IndexCdsQuotes`, `Data.IndexCdoQuotes`,
  `Data.IndexCdsSpreadQuotes` and `Data.IndexCdoSpreadReferenceQuotes`,
  and `IndexCds.makeStandard` and `IndexCdo.makeStandard` take it.
  `Data.IndexCdsKey` only identifies the version. The constituents' weights of an
  `IndexCds` or `IndexCdo` add up to 1: each name is 1/124, not 0.008,
  once 1 of 125 has defaulted. Tranche points and `Data.BaseCorrelation`
  detachments are fractions of the version's outstanding notional:
  (X - L) / (1 - w) for a point X of the previous version, w the
  defaulted weight and L its loss, the senior tranche detaching at 100%.
  At 30% recovery on 1 of 125 names, 0-3% becomes 0-2.4597% and 3-7%
  becomes 2.4597-6.4919%. `IndexCds.applyCreditEvent`,
  `IndexCdo.applyCreditEvent`, `BaseCorrelation.applyCreditEvent` and the
  three `applyCreditEventTo...Quotes` functions give the next version in
  these units, and everything that takes a version reads them: the index
  strip, the index and tranche pricers, base correlation calibration and
  mapping, `impliedBaseCorrelation` and the option pricers.
  `indexFactor()` on an index or tranche is now its `indexFactor`
  attribute, and `outstandingFactor()` the factor times the weights' sum
  (which differs for a part of a version-1 index). The market still names
  a tranche by its points at version 1, its quoted points:
  `Data.IndexCdo` carries `quotedAttachment` and `quotedDetachment`, and
  `Data.IndexCdoQuotes` (and the spread reference quotes)
  `quotedAttachments` and `quotedDetachments`, required arguments right
  after the points (and after them in `IndexCdo.makeStandard`), the
  points themselves while the index factor is 1. `makeV1` on each of the
  three makes one on version 1 with the quoted points and factor filled
  in. The tranche factor, `trancheFactor()` (`trancheFactors()` on
  the quotes), is the outstanding notional per unit of quoted notional:
  the width times the index factor over the quoted width - in Markit's
  CDX.NA.HY 33 example, 0.9395 for the 0-15% equity and 0.99858 for the
  35-100% super senior. The credit events keep the quoted points and set
  the factor. A trade's notional is its current notional, which its legs
  are on: `IndexCds.makeStandard` is given the original notional and
  applies the index factor, and `IndexCdo.makeStandard` (and
  `generateIndexCdo`) the quoted notional and applies the tranche factor -
  10m quoted at a tranche factor of 0.8133 is a trade of 8.133m. Before,
  `IndexCdo.makeStandard` took the current notional; calls after version
  1 must give the quoted one. Upfronts and deltas stay
  per unit of each tranche's and index's own outstanding notional, and an
  index option's `settledLoss`, prices and exercise price stay per unit of
  original notional. Values and risks are unchanged: tests price each
  next version both ways, in the market's convention and in the original
  notional's units, and get the same value. Bespoke tranches keep their
  pool's original notional.

  This breaks objects made after version 1 in the previous convention. An
  `IndexCds` or `IndexCdo` after version 1 whose weights do not add up to
  1 is refused, so one saved by 0.3.0 fails to load rather than pricing
  wrongly. Make it again with its index factor, its weights over the
  factor, its points over the factor and its quoted points. Code making a
  tranche or tranche quotes passes the quoted points after the points, or
  uses `makeV1` at version 1. The notes on the
  quote conversions now say when the next version trades: once the
  Determinations Committee has announced the event, around its event
  determination date, for the events covered (not restructuring).
- **An index option quote sheet keeps the version its options were traded
  on.** Options keep referencing their version for their whole life, and
  dealers keep quoting them on it after a name defaults.
  `CreditOption.applyCreditEventToIndexCdsOptionQuotes` now takes only the
  sheet, the index as its options stand and the event. It keeps the
  sheet's key, strikes, prices, forward and reference as quoted, adds the
  names' loss to its `settledLoss` as for a traded option, and gives the
  index's next version. It no longer converts the reference, and accepts
  a sheet observed on any date. `settledLoss` is now the loss since the
  key's version. `impliedIndexCdsVolatilities` and the functions that
  take a sheet accept an index of the sheet's version or a later one: the
  current version, as the same events leave it, on the names left's
  curves. `impliedIndexCdsOptionQuotes` takes `quotesKey`, the version the
  options were traded on, after `settledLoss`. A sheet saved after an
  event on the next version's key no longer fits its options: apply the
  events to the sheet as quoted. Values are unchanged.
- **`Data.BespokeCds` takes a fixed recovery for each contingent leg.**
  `fixedRecovery` is empty, or has one recovery for each contingent leg, in
  the order of `contingentLegs`. Each leg pays one minus its own recovery,
  in `CreditCorrelation.pvBespokeCds`, its greeks and
  `Data.BespokeCds.applyCreditEvent`. Before, `fixedRecovery` had at most
  one entry, which applied to every leg. A swap with one contingent leg is
  unchanged. A swap saved by 0.3.0 with one fixed recovery and several
  contingent legs no longer loads: give the recovery once for each leg.
- **`Curves.termRateHedges` takes the overnight index.** `overnightIndex`,
  the third input, is the currency's overnight index, the one the OIS strip
  given to `Curves.termRateStrip` was built on. The overnight leg of each
  basis swap hedge names it, so a book that holds the hedges knows which
  index's curve and fixings that leg takes. Before, the leg named no index.
  Calls must add it after `index`.
- **`Data.FxRate` has an observation date.** `observationDate`, its first
  input, is the date the rate is observed on, and its spot date is
  `spotDays` business days after it. Before, the spot date was taken from
  the discount curves' base date. Each FX pricer, strip, hedge and
  cashflow function now refuses a rate observed on another date than its
  curves' or quotes' base date. A fixing applied to a trade must be
  observed on the trade's fixing date: `Data.FxForward.applyFixing`,
  `Data.FxVanillaOption.applyExpiry` and `Data.CrossCurrencySwap.applyFxReset`
  (on `resetDate`). `Data.CrossCurrencySwap.makeStandard` and
  `Curves.fxSpotHedge` take a rate observed on their trade date. Calls
  must add the date before `baseCurrency`.

### Added

- **Quote types in flat spreads.** `Data.CdsSpreadCurve` is a name's CDS
  curve quoted in spreads: the fields of `Data.CdsCurve`, with one running
  coupon and a flat spread per quote in place of each quote's coupon and
  upfront. `Data.IndexCdsSpreadQuotes` is `Data.IndexCdsQuotes` with
  spreads in place of upfronts, and `Data.IndexCdoSpreadReferenceQuotes`
  is `Data.IndexCdoQuotes` with `referenceIndexSpreads` in place of
  `referenceIndexUpfronts`, under the same rules. None carries a recovery
  rate: the conversions are given the one the spreads are quoted with, so
  it is recorded once with the rest of the market data.
- **`IsdaCds.indexCdsQuotesToSpreads` gives a CDS index's quotes as an
  `IndexCdsSpreadQuotes`.** It is the inverse of
  `IsdaCds.indexCdsQuotesFromSpreads`, with the same recovery rate and
  discount curve: each upfront converted with the index's coupon on the
  series' maturity, the index priced as a single CDS, as CDSW quotes
  iTraxx and CDX. The key's index factor does not enter, since a version's
  upfronts are per unit of its own outstanding notional, after a credit
  event as before: a version kept in spreads converts to upfronts, takes a
  credit event with `Curves.applyCreditEventToIndexCdsQuotes`, and
  converts back. On the quotes `Curves.impliedIndexCdsQuotes` gives on the
  names' own curves, it gives the index's intrinsic spreads, and the index
  basis in spread.
- **The SPI log tools ship with qeLib.** `qeLib.log_tools()` says where
  they are for the running Python: `spi_log_to_python.py`, which turns an
  SPI function log into a Python script replaying every call
  (`--with-checks`: each result against the logged one, OK/DIFF/ERROR),
  `spi_log_to_excel.py`, which turns it into an Excel workbook, and
  `spi_log_to_excel_launcher.py`, which opens that in Excel with the
  add-ins (`--addins DIR`), in front of the window that asked for it, as
  `spi_log_to_excel.exe` does too - with `spi_replay`, which they read logs with,
  and the services' `.svo` files. The wheel carries them as
  `qeLib/log_tools`; the Excel zip's `spi_log_to_excel.exe` is as it was.
  The Python script needs nothing more; the workbook needs `openpyxl`, and
  opening Excel `pywin32`.

- **Credit event settlements as of the end of the valuation date.**
  `Data.ValuationContext` has a fourth flag, `eodCreditSettlements`, true by
  default. It says whether a credit event whose auction settlement date is
  the valuation date has already settled. If it is false, the event is
  still to settle, and quotes observed on that date are still the previous
  version's. `Curves.applyCreditEventToIndexCdsQuotes`,
  `Curves.applyCreditEventToIndexCdoQuotes` and
  `CreditOption.applyCreditEventToIndexCdsOptionQuotes` take an optional
  `valuationContext` and then convert quotes observed on the settlement
  date. The slice the event settles that day is paid in full. Quotes
  observed after the settlement date are still refused. With no context
  given, or the flag true, the results are unchanged. A trade's settlement
  cashflows on the valuation date follow `eodPayments`, as before.
- **A SABR smile per expiry for index CDS options.**
  `CreditOption.calibrateIndexCdsSabr` fits alpha, rho and nu to each
  expiry's quoted volatilities (`Data.IndexCdsVolatilities`), with beta
  given (1 by default, the lognormal spread). It gives an
  `IndexCdsSabrCalibration`: the smiles, the strikes as spreads, the fit's
  errors, and the Jacobian of the parameters to the quoted volatilities.
  The smiles, a `CreditOption.IndexCdsVolatilityCalculatorSabr`, are a
  volatility calculator for `pvIndexCdsOption`, `greeksIndexCdsOptions`
  and `indexCdsOptionDensity`, an alternative to
  `IndexCdsVolatilityCalculatorInterpolated`.
  - SABR is on the default-adjusted forward spread with the front-end
    protection, the forward the pricer prices at, held at the
    calibration's: the smile is sticky strike. Price strikes are converted
    to spreads as the pricer settles them. The volatilities are Hagan's
    lognormal expansion's.
  - Between expiries, the total implied variance is linear in time at a
    fixed moneyness; before the first and after the last, each one's smile
    applies.
  - The smile is smooth in the strike, so its density has no spike or step
    at a quote, and no wing to extrapolate. On quotes a little off a smooth
    smile, where linear or monotone cubic interpolation gives a negative
    density (a butterfly arbitrage), the SABR fit's is positive.
  - In the LognormalExchange convention the smile is a parametrisation of
    that convention's volatilities in the strike over the forward spread.
  - The smiles' parameters are alpha, rho, nu and the forward, for each
    expiry. The greeks to them project back with `Data.Greeks.project` and
    the calibration's `jacobian()`: to the quoted volatilities, and through
    the forwards to the survival, recovery and discount curves
    (`curveJacobian`). The curve greeks are then those of revaluing on a
    smile recalibrated to the same quotes. Without the projection, holding
    the smile, they differ by 0.2% to 0.8% of the curve delta near the money
    and 6% in the wing in the tests.

- **CDS and index CDS quotes implied by the curves.** The inverses of the
  credit strips, as `Curves.impliedInterestRateSwapCurve` is the OIS
  strip's.
  - `Curves.impliedCdsCurve` gives a `Data.CdsCurve` with each quote's
    upfront set to the one a survival curve implies. At that upfront, the
    quote's standard CDS, as the strips make it, reprices on the survival,
    recovery and discount curves. The curve given is a template: its
    maturities, coupons and conventions are kept, and its upfronts are not
    used.
  - With `par`, `Curves.impliedCdsCurve` sets each coupon to its par spread
    instead, with no upfront. The par spread is exact, since the upfront is
    linear in the coupon. A quote whose upfront does not fall with its
    coupon has no par spread, and the function says which quote it is.
  - `Curves.impliedIndexCdsQuotes` gives a `Data.IndexCdsQuotes` with each
    upfront set to the one the constituents' curves imply, for the index
    CDS that `Curves.indexCdsStrip` makes.
  - Stripping quotes and then implying them on the strip's curves gives
    them back, to the strip's tolerance. On the names' own, unadjusted
    curves, the index's implied upfronts are its intrinsic upfronts.
  - Other curves imply the quotes they stand for: for example slid,
    forward or bumped curves, or a credit event scenario's curves.

- **Swaption and cap volatility quotes implied by a model.**
  `RatesOption.impliedSwaptionVolatilitiesTermRate` and
  `impliedSwaptionVolatilitiesOis` take a `Data.SwaptionVolatilityQuotes`
  as a template, with any `SwaptionModel`, e.g. a SABR calibration's
  `model()`. They return the quotes with each volatility set to the one at
  which the quote's swaption has its premium in the model.
  `impliedCapVolatilitiesTermRate` and `impliedCapVolatilitiesOis` do the
  same for a `Data.CapVolatilityQuotes`, with any `CapletModel`: stripped
  caplet volatilities or SABR.
  - The volatilities are quoted as the template is, normal or lognormal at
    its shift, whatever the model's own quoting. The template's
    volatilities are not used.
  - Each quote's option is the one that the premium conversions and the
    calibrations take. The premium is a straddle's, so the volatility is
    the same whichever way the forward is off the strike.
  - The round trip holds. The caplet volatilities stripped from cap quotes
    imply them back, and a cube's own Black-Scholes model implies the cube.
    A SABR calibration implies its `modelVolatilities`, the quotes to
    within its fit. Calibrated again to what it implies, it implies the
    same.

- **FX volatility quotes implied by a surface.**
  `FxOption.impliedFxVolatilityQuotes` is the inverse of
  `calibrateFxVolatility`. It takes a `Data.FxVolatilityQuotes` as a
  template, with any `FxVolatilityCalculator`. At each expiry it returns
  the at-the-money volatility, the 25 delta risk reversal and butterfly
  and, if the template has them, the 10 delta ones of the surface's smile.
  - They are in the template's conventions: its delta types, at-the-money
    conventions and butterfly type. The template's values are not used.
  - The at-the-money volatility is the one at which the at-the-money
    strike, by its convention, has the smile's volatility. A broker
    butterfly is the volatility at which the strangle, struck at that
    volatility's own delta strikes, has the smile's price, less the
    at-the-money volatility. A smile butterfly's wings are found the same
    way, each at its own delta strike.
  - With `quoteTime`, the surface must be as of that time. The quotes are
    then given on whole days, as the market quotes them and as the
    calibration takes them.
  - The round trip holds. A calibration to 25 delta quotes meets them, and
    its surface implies them back: for SABR, Vanna-Volga and SVI, broker
    and smile butterflies, and spot and premium-adjusted deltas. With 10
    delta quotes the fit misses them; the quotes its surface implies, fitted
    again, are met and implied again.

- **Index CDS option volatilities implied by a surface.**
  `CreditOption.impliedIndexCdsVolatilitiesFromSurface` is the inverse of
  `calibrateIndexCdsSabr` and of `IndexCdsVolatilityCalculatorInterpolated`.
  It takes a `Data.IndexCdsVolatilities` as a template, with any
  `IndexCdsVolatilityCalculator`. Each quote is set to the volatility at
  which its option has the price it has on the surface.
  - The price is the out-of-the-money option's, the payer at strikes at or
    beyond the expiry's forward, as `impliedIndexCdsVolatilities` takes it.
  - The surface's convention and the result's are given separately, so
    the function also converts between LognormalSpread and
    LognormalExchange through prices. The template's strike quoting is
    kept, whatever the surface's.
  - The round trip holds. An interpolated surface implies its quotes back,
    in either convention and strike quoting. A SABR calibration implies its
    `modelVolatilities`; calibrated again to those, it meets them and
    implies them again.

- **Base correlations implied by a loss distribution.**
  `CreditCorrelation.impliedBaseCorrelation` takes a `Data.BaseCorrelation`
  as a template, with any `LossDistribution` of the pool. It returns each
  entry's base correlation: the one at which the Gaussian copula's base
  correlation distribution gives the equity tranche to the entry's
  detachment the expected loss the distribution gives it by its date.
  - It is `calibrateBaseCorrelationToExpectedLosses` on the distribution's
    expected losses. So each entry depends on its own date and detachment
    alone, whatever interpolation the result is priced with.
  - For a copula, e.g. NIG or RFL, the result is its base correlation
    skew. A Gaussian copula's is flat at its correlation. For base
    correlations interpolated or mapped from another pool, it is their
    values at the template's points.
  - The template's correlations are not used. An entry detaching at the
    pool's notional takes the base correlation of the highest detachment
    below it at its date, as `calibrateBaseCorrelation` has it.
  - The round trip holds. A base correlation distribution implies its own
    base correlations back, at its points and between them. The skew a
    NIG copula implies gives back the copula's expected losses.

- **FX options are valued at a time of day.** `FxOption.pvFxVanillaOption`
  and `FxOption.greeksFxVanillaOptions` take an optional `valuationTime`, a
  `Data.TimeOfDay`, applied on the curves' base date in its own time zone.
  - `Data.FxVanillaOption` has an optional `expiryCut`, a `Data.TimeOfDay`:
    the time of day at which it expires and, if cash-settled, at which its
    fixing is taken. `makePhysical` and `makeCash` take it last. An option
    without a cut expires at the New York cut, 10:00 America/New_York.
  - `Data.TimeOfDay` is a time of day local to a time zone, on no
    particular date: `hour`, `minute` and `timeZone`, an IANA name, UTC if
    omitted. The date it applies on is the context's: a pricer's valuation
    date, an option's expiry date. Its zones keep their current daylight
    saving rules for every year, so 10:00 New York is 15:00 UTC in winter
    and 14:00 UTC in summer. It is not FX-specific. `FxOption.fxCut` gives the
    market's standard cuts, named by the enum `FxOption.FxCut`: NewYork,
    10:00 New York; Tokyo, 15:00 Tokyo; and ECB, 14:15 Frankfurt. Any
    other cut, e.g. a local fixing's at 13:00 America/Sao_Paulo, is a
    `Data.TimeOfDay` of its own. Asia/Seoul, Asia/Taipei, Asia/Manila and
    Asia/Jakarta are among its zones, for the KRW, TWD, PHP and IDR NDO
    fixings.
  - The forward and the discounting stay on the dates. The variance runs
    from the valuation time to the cut: the surface's variance in
    proportion to the volatility time left, after Jäckel's time-weighted
    volatility. A weekend or a holiday takes little of it if the surface's
    days weigh so (`Data.FxVolatilityTimeWeights`), an event day more, and
    within a day it runs evenly.
  - On the expiry date the option keeps the variance left until the cut,
    and is worth its intrinsic value from the cut on. Before, it was worth
    its intrinsic value all day. `expiryTime` and `volatility` in
    `FxOption.FxVanillaOptionPvResult` are now exact to the second.
  - `FxOption.calibrateFxVolatility` takes an optional `quoteTime`, the
    `Data.TimeOfDay` the quotes were taken at on the valuation date, and
    `quotesCut`, a
    `Data.TimeOfDay` (the New York cut if omitted). The market quotes on whole days at any time of
    day, so each smile is fitted at the exact time from `quoteTime` to its
    cut, to quotes scaled to keep each option's variance. The model values
    and the Jacobian stay in the quotes' own terms. The surface is as of
    `quoteTime`: `FxOption.FxVolatilityCalculatorInterpolated` has it as
    `quoteTime`, and is otherwise as of the New York cut on its valuation
    date.
  - `Data.FxVolatilityTimeWeights` takes optional `intradayWeights`, 24
    weights for the hours of the UTC day (more in the London and New York
    sessions, say). They are scaled to average 1, so each day keeps its
    weight and only how it runs through the day changes. It also takes
    `timedEventDates`, `timedEventTimes`, each a `Data.TimeOfDay`, and
    `timedEventWeights`: an
    event at a time, such as the payrolls at 08:30 New York, adds its
    weight less its day's usual weight at that instant, and that variance
    is gone once it has happened. The events in `eventDates` have no time,
    and their weight is spread over their day. Whole days weigh as
    before.
  - `FxOption.FxVolatilityCalibration` keeps the `quotesCut` it was given,
    None for the New York cut, and `FxOption.fxVanillaOptionHedges` gives
    each hedge that cut. Before, the hedges always expired at the New York
    cut, so at a valuation time the hedges of quotes on another cut were
    not the quoted options. Take the hedges' greeks at the book's
    `valuationTime`.
  - `FxOption.fxRateDistribution` takes an optional `valuationTime` and
    `expiryCut`, each a `Data.TimeOfDay`: the rate's distribution as seen
    from a time of day, its variance to the cut as the pricer has it, so
    its call values are the pricer's PVs at that time. Without a valuation
    time it is on dates, as before.
  - Without a valuation time every result is unchanged, with or without a
    cut on the option. So is a calibration without a quote time. So is
    every result without timed events or hours' weights.
- **Each reported cashflow has a type.** `Data.CashflowListReported` has
  `types`, a `Data.CashflowType` for each cashflow, which says what the
  cashflow pays:
  - `Coupon`: fixed, floating, CMS and range accrual coupons, CDS and
    tranche premium coupons, and bond coupons.
  - `Notional`: cross currency swap notional exchanges, the payments of a
    notional payment leg, and the cash lent at a repo's start.
  - `Redemption`: a bond's principal, a bill's face amount, and a repo's
    repayment.
  - `Settlement`: a bond's or bill's settlement amount, and a cash-settled
    FX forward's net amount.
  - `Exchange`: the currencies a physically settled FX forward or FX
    option delivers.
  - `Exercise`: a cash-settled option's payoff, from a caplet or floorlet,
    a CMS cap or floor, a cash-settled swaption or FX option, or an index
    CDS option.
  - `Other`: the cashflows of a `Data.CashflowList`, which carry no type.

  A physically settled swaption or Bermudan exercised reports the swap's
  cashflows with their own types. In Python `types` is a list of strings,
  like `statuses`, and in Excel it is
  `qeLib.Data.CashflowListReported.types`. The dates, values and statuses
  reported are unchanged. `types` is optional when a list is made
  directly: omitted, each cashflow is `Other`, so a list saved by 0.3.0
  still loads.
- **`Curves.fxRateForward`, `Curves.fxRateSlide` and
  `Curves.fxRateAnchorBack` move a spot rate to another date.**
  `fxRateForward` gives the forward for a later date's spot date, observed
  on that date. `fxRateSlide` keeps the rate and moves its observation
  date. `fxRateAnchorBack` takes a rate observed on a later date back to
  the curves' base date: the rate whose forward for the later spot date is
  the rate given. They are a theta's moves of the spot rate, as
  `Curves.DiscountCurveForward`, `Curves.DiscountCurveSlide` and
  `Curves.DiscountCurveAnchorBack` are of a curve.
- **`Curves.cdsStripFlatHazardMakeRobust` strips CDS quotes that
  `Curves.cdsStripFlatHazard` fails on.** It keeps every segment's hazard
  rate between `limitMargin` and 10 less `limitMargin`, so that the
  resulting quotes can be bumped in a risk report and stripped again.
  Where no hazard rate in that range reprices a quote, it puts the
  segment's hazard rate at the end of the range the quote is beyond, and
  corrects the quote's upfront to the one that hazard rate gives. A quote
  that strips, but to a hazard rate less than `limitMargin` from 0 or 10,
  is corrected as well. It ends the curve at the first pillar
  where the survival probability is `minSurvival` or less, and leaves the
  later quotes out. It returns a `Curves.CdsStripFlatHazardRobustResult`:
  the survival curve, the resulting upfronts, each quote's status
  (`Matched`, `Lower`, `Upper` or `CutOff`), and the corrected CDS curve,
  which `Curves.cdsStripFlatHazard` strips to the same survival curve.
  With nothing corrected or left out, the survival curve is exactly
  `Curves.cdsStripFlatHazard`'s.
- **`Curves.cdsStripInterpolatedCumulativeHazardMakeRobust` does the same
  for `Curves.cdsStripInterpolatedCumulativeHazard`.** It starts from the
  curve of `Curves.cdsStripFlatHazardMakeRobust`, but decides for itself
  which quotes to correct and where the curve ends, since a flat hazard
  rate may not reach quotes the interpolation does. Should that start
  leave quotes corrected, it also solves from every segment at
  `limitMargin`, and keeps the result that corrects fewer. It solves the
  pillars together.
  A pillar is pinned `limitMargin` inside its limit, and its quote
  corrected, where repricing the quote would need a hazard rate over its
  segment below `limitMargin` or above 10 less `limitMargin`. A pinned
  pillar is freed again if its quote
  turns out to be within reach. If the solver stalls, the quote with the
  largest error is given up, with the status `Unmatched`, so the function
  always returns a curve. A pinned quote the curve still reprices is
  `Matched`. It returns a
  `Curves.CdsStripInterpolatedCumulativeHazardRobustResult`, with the same
  fields as the flat hazard one. It is idempotent: given its corrected CDS
  curve, it matches every quote. `LinearAverageHazard` can still give a
  negative hazard rate between pillars.
- **`IsdaCds.cdsStripRobustSpreadReport` reports a robust CDS strip in
  flat spreads.** For a curve quoted in spreads and converted by
  `IsdaCds.cdsCurveFromSpreads`, it takes the CDS curve and the `upfronts`
  and `status` of the robust strip's result, and returns an
  `IsdaCds.CdsStripSpreadReport`: each quote's quoted spread, the spread
  the stripped curve gives, its status, and a message for each quote that
  was not stripped as quoted, e.g. `5Y: the quoted 300.00bp is below what
  the other quotes allow. 557.14bp was used.` `summary()` gives the
  messages, one per line.
- **`IsdaCds.cdsCurveToSpreads` gives a CDS curve's quotes as flat
  spreads.** It is the inverse of `IsdaCds.cdsCurveFromSpreads`, with the
  same recovery rate and discount curve.
- **The tranche greeks give each name's jump to default.**
  `CreditCorrelation.greeksIndexCdos` and `CreditCorrelation.greeksBespokeCdos`
  give `jumpToDefault` when `inputs` names it: each trade's value with a
  name of the pool defaulted now, less its value, with a row for each name.
  The name is certain to have defaulted by every loss date, at its fixed
  recovery or its recovery curve's at the base date. The other names, the
  curves and the base correlations at today's points are held. The
  expected losses with each name defaulted come from the sweep that gives
  the loss distribution's sensitivities: each name is removed from the
  conditional distributions and added back with certain default. Each trade
  is then priced on them once per name. The result is the same as
  repricing with the name's survival curve defaulted at the valuation date.
  A credit event applied to the trade differs, because it settles the loss
  on its own date and moves the points the base correlations are read at.
  `jumpToDefault` is not a derivative, so it is left out when `inputs` is
  omitted, and `Data.Greeks.project` leaves it as it is. A distribution
  that can't default its names, such as a
  `CreditCorrelation.LossDistributionExpectedLosses`, fails.
- **`Curves.pvIndexCdsSubstitutions` and
  `CreditOption.pvIndexCdsOptionSubstitutions` price one name's curves
  substituted at a time.** Each takes the names' positions and a
  substitute survival curve for each, and optionally a substitute recovery
  curve. It gives the change in price from each substitution, with the
  other names held. The index CDS is a sum over its names, so only the
  substituted names' CDSs are priced. The option depends on the names only
  through its default-adjusted forward, so the forward, the strike
  conversion and its table of upfronts are built once. Each substitution
  then replaces its name's part of the forward and prices the option again
  on the same conversion. Both agree with repricing with the curves
  replaced, to rounding. A name's survival curve defaulted now,
  `Curves.SurvivalCurveFutureDefault` at its base date, gives its jump to
  default.
- **`RatesOption.SwaptionModelShocked` and `RatesOption.CapletModelShocked`
  shock a rates model's density.** A `RatesOption.RateDensityShock` is a sum
  of bumps. Each adds its amount, times a cubic B-spline in the strike, to
  every distribution's undiscounted call and put values. Its centre is an
  offset from each rate's forward, or a strike if absolute, and it reaches
  its width either side. The density changes by the bump's curvature: a
  positive amount moves probability out from the centre to either side.
  The probability and the forward are unchanged, so put-call parity holds.
  Every pricer on the shocked model sees the shocked density: swaptions,
  caps, floors, range accruals, CMS coupons through their replication, and
  the Markov functional model built on it. The amounts are the model's
  last parameters, "density shock 1" and so on, so a book's greeks to the
  model include each bump.
  With `units` set to `StandardDeviations`, the centres and widths are
  multiples of each distribution's at-the-money standard deviation, the
  square root of 2 pi times its at-the-money call value. That is a normal
  rate's standard deviation exactly. So one shock lands at the same
  moneyness at every expiry and tenor. With `amountType` set to
  `NormalVolatility`, an amount is a change of normal volatility at the
  bump's centre. The bump then adds Bachelier's vega there times the
  amount, the vega taken over the option's variance time, so the greeks
  to the amounts are vegas. The shock does not keep the density positive by
  itself: `RatesOption.swaptionRateDistribution` and
  `RatesOption.capletRateDistribution` show the shocked density.
- **`RatesOption.RateDensityShock.makeProfile` and `makeSmile` give a shock
  as the change it makes to the smile.** `makeProfile(points, volChanges)`
  changes the normal volatility by `volChanges[i]` at `points[i]`, in
  at-the-money standard deviations from the forward, at every expiry and
  tenor, to first order. It centres one bump on each point, as wide as the
  distance between its neighbours, and solves for the amounts. The change
  is smooth between the points and fades to nothing two spacings beyond the
  outer ones. `makeSmile(level, skew, curvature, points?)` is the profile of
  level + skew x + curvature x^2, at -2, -1, 0, 1, 2 unless given. Both
  return an ordinary shock, so its parameters are the bumps' amounts.
- **`RatesOption.RateDensityShock.profileJacobian` and `smileJacobian` take
  a shocked model's greeks to the profile or the smile.** They give how
  the bumps' amounts move with the volChanges, or with the level, skew and
  curvature: a fixed matrix, the amounts being linear in them. Its input
  is named as the greeks name the model, `model` unless given, with the
  parameters "vol change at x SD", or "level", "skew" and "curvature".
  `Data.Greeks.project` with it gives a book's vegas to each point, or its
  level, skew and curvature vegas, beside the model's other parameters.
- **`Curves.greeksIndexCdss` and `CreditOption.greeksIndexCdsOptions` give
  each name's jump to default** when `inputs` names `jumpToDefault`, as the
  tranche greeks do: the value with the name's survival curve defaulted
  now, less the value, with a row for each name. They come from the
  substitution functions above. A payer option gains on any name's
  default, since the loss goes into the front-end protection.

### Changed

- **Black and Bachelier implied volatilities by Jäckel's methods.** They
  were implied by Brent's method on the price. Now lognormal ones use
  "Let's be rational" (2016, after "By implication", 2006) and normal ones
  his analytic formula ("Implied normal volatility", 2017). The swaption
  SABR calibrations use them: for their quote conversions, and for
  Antonov's values.
  - The numerics are Jäckel's reference implementations, vendored in the
    Math library with his notice (`licenses/NOTICES.txt`). qeLib takes
    ln(F / K) by log1p near the money: the reference's log(F / K) loses
    up to half the digits of a small implied deviation there.
  - Each volatility is as accurate as the price's digits allow. Against
    prices to 60 digits, over 4,100 use-case and stress points, 99% are
    within 2 units in the last place of that limit, and all but six
    within 5. Brent's method stopped at 1e-12 in the standard deviation
    and was 10 to 90 units off at the median.
  - It took any price within 1e-14 of the intrinsic value to have no
    volatility, out of the money too. So small premiums of deep
    out-of-the-money, short-dated or low-volatility options implied 0.
    Now only an in-the-money price within the forward's and the strike's
    rounding of the intrinsic value does. That is about ten times less,
    and it is why six points, with a time value of a few units in the last
    place, still imply 0.
  - They are faster: about 350 to 400 ns for a Black volatility against
    1,100 to 1,400 ns, and 70 to 100 ns for a normal one against 600 to
    750 ns.
  - `Math::LetsBeRational` also gives the normalised Black function. It is
    accurate where the plain formula loses digits: near the money at small
    standard deviations, and far from it.

- **`Math::NormalDistribution`'s cdf and inverse cdf are Jäckel's too.**
  The cdf is Cody's rational erfc, from the same vendored code, with an
  asymptotic series below -38.5. The inverse is Jäckel's "PJ-2024"
  rational approximations, which need no iteration, in place of Acklam's
  approximation refined by two Halley steps. Against 60-digit references:
  - The inverse is within 3 units in the last place everywhere, from
    p = 1e-300 to 1 - 1e-15. The old one lost up to five million units near
    p = 1/2, where its refinement against the cdf cancels. It takes 7 ns
    against 55 ns.
  - The cdf is as accurate as before where the copulas use it, x in
    [-8, 8]. There the rounding of exp(-x^2 / 2) that x's own rounding
    brings is the limit for both. Below -37 the old one underflowed into
    subnormal numbers and lost all its digits. It takes 9 ns against 10 ns.
  - The tranche, nth-to-default, bespoke and pool analytics tests take the
    same time as before, within 1%. The copulas cache each name's
    threshold, and building the loss distributions is most of the time.
  - The derivatives the AAD overloads record are unchanged: the density
    for the cdf, and its reciprocal for the inverse.

- **`Curves.cdsStripInterpolatedCumulativeHazard` and
  `Curves.cdsStripInterpolatedCumulativeHazardMakeRobust` share one
  solver.** The plain strip had its own Newton solve from
  `Curves.cdsStripFlatHazard`'s curve, and threw where that strip failed.
  It is now the robust strip's solver, with every segment's hazard rate
  kept within 0 and 10 and the curve never ended, and throws, naming the
  quote, where the robust strip would correct one. So the two agree: where
  the plain curve keeps the robust strip's margins, the robust strip
  returns it bit for bit, and the plain strip strips the robust strip's
  corrected CDS curve. A monotone cubic may reprice a short quote above
  what a flat hazard rate of 10 gives, by putting more of a segment's
  hazard early, so the plain strip now strips some quotes it threw on.
  Its curves may differ slightly from before, as the solve takes other
  steps to the same quotes.
- **A tranche certain to be wiped out is priced as wiped out under base
  correlations too.** The tranche pricers take a tranche's surviving
  notional as 0 from 1e-12 down, where before it was from 1e-15. Under base
  correlations a tranche's expected loss is a difference of two equity
  tranches over its width, and rounds to several times 1e-15. So a tranche
  certain to be wiped out could come to just above 1e-15 and be priced as
  decaying over its first loss period, rather than as wiped out at the
  valuation date. That moved a tranche's value by as much as 2bp of its
  notional, depending on rounding.

- **The SABR calibrations and the caplet strip have a Jacobian to the
  curves.** `RatesOption.SwaptionSabrCalibration`,
  `RatesOption.CapletSabrCalibration` and
  `RatesOption.CapletVolatilityStripping` have a `curveJacobian`: how the
  calibrated parameters move with the curves they were calibrated on, with
  the volatility quotes held. Its inputs are `discountCurve`, and
  `projections` or `projectionCurve`, named as the greeks functions name
  them. It comes from the implicit function theorem on the same fits as
  `quoteJacobian`. For the swaption cube, it goes through each node's
  forward, which moves the strikes with it. For the caps, it goes through
  their prices in the model, their target prices and their vegas, so
  through the forwards and the discount factors alike. Their `jacobian()`
  now has these inputs as well as `volatilityQuotes`. So
  `Data.Greeks.project` adds the risk that comes through the parameters
  to the greeks to the curves, which then hold the volatility quotes and
  not the parameters. Before, the risk to the curves held the parameters.
  A parameter that a fit leaves at a bound, such as nu at 0 or rho at
  ±0.999, is held in both Jacobians. A calibration saved before has no
  `curveJacobian`, and its `jacobian()` is to the quotes alone.

- **`Data.OvernightCompoundedLeg` takes an observation shift with no
  lookback.** Before, `observationShift` without `lookbackDays` was refused.
  A shift of no days moves nothing, so the coupon is as without it. The
  make functions that build the leg, such as `Data.OisSwap.makeStandard`,
  take it too.
- **`Curves.fxForwardCashflows` takes `fxRate` and the discount curves
  when they are needed.** Before, it refused some of them without the
  others. Only a cash-settled forward that pays in the window needs them,
  all three. Otherwise they may be given or left out.

### Fixed

- **`RatesOption.calibrateCapletSabrTermRate` and
  `RatesOption.calibrateCapletSabrOis` fit a segment after one whose rho
  reached its bound.** Each maturity's smile is fitted from the previous
  maturity's rho. Where that fit ended with rho at -0.999 or 0.999, as a
  skew flatter than beta's own leads to, the next fit started from an
  infinite variable and threw "LevenbergMarquardt::minimize: start must be
  finite". It now starts from a rho of ±0.95 there, on the same side, and
  may go back to the bound. Likewise, a previous alpha of 0 gives way to
  the first segment's estimate from the quotes. Calibrations that did not
  throw are unchanged.

## 0.3.0 - 2026-11-13

### Interface changes

- **`Data.Cds`, `Data.NthToDefault`, `Data.IndexCdo`, `Data.BespokeCdo` and
  `Data.BespokeCds` no longer keep holiday calendar names.** Nothing used
  them: these instruments' dates are already adjusted. Their constructors
  lose their last input, `holidays`, and the types the field. The
  `makeStandard` functions still take the names and the calendars, to
  adjust the dates. Objects saved by 0.2.0 still load, without the names.
  `Data.IndexCds` keeps its names, which options on it use for their
  settlement and strike conversion.

## 0.2.0 - 2026-11-12

### Interface changes

These may need changes to workbooks and scripts written for 0.1.0.

- **Six Data types are built only by their make functions.** Their
  constructors took optional fields that had to be set in combination, or a
  swap of a kind they then refused. Each make function takes only arguments
  that are required, or optional on their own. The types' fields can still be
  read, and objects saved by 0.1.0 still load.
  - `Data.InterestRateSwap`: `makeFixedAgainstTermRate`,
    `makeFixedAgainstOvernight` and `makeTermRateAgainstOvernight`, as well
    as `makeStandard` and `makeBasisSwap`. A swap with all three kinds of leg
    can no longer be made.
  - `Data.CmsSwap`: `makeFixed`, `makeTermRate` and `makeOvernight`, one for
    each kind of funding leg, as well as `makeStandard`.
  - `Data.CrossCurrencySwap`: `makeConstantNotional` and `makeResetting`, as
    well as `makeStandard`.
  - `Data.TermRateStub`: `makeSingle` and `makeInterpolated`.
  - `Data.SwaptionTermRate` and `Data.BermudanSwaptionTermRate`: `make`, as
    well as `makeStandard`. `make` takes the strike, the notional, the fixed
    leg's day count and dates, and the term rate leg, in place of a swap.
- **`RatesOption.pvInterestRateLegsInstrument` and
  `RatesOption.greeksInterestRateLegsInstruments` take two more optional
  inputs**, `capletModels` and `rangeAccrualModel`, after `cmsModels`, for
  range accrual legs. In Excel, and in Python calls that pass inputs by
  position, `overnightProjection`, `calendars` and `valuationContext` move
  two places along.

### New

- **A Python wheel for macOS** 11 and later, with Python 3.14: one universal2
  wheel, for Apple Silicon and Intel Macs.
- **Range accrual legs**, `Data.RangeAccrualLeg`, which pay each coupon on
  the share of its days on which a rate is in a range. They are built from
  three parts: the coupons (`Data.RangeAccrualCoupons`, fixed or floating),
  the range (`Data.RangeAccrualRange`, between, above or below barriers) and
  what is observed (`Data.RangeAccrualObservation`, a term rate, an overnight
  rate or a swap rate). They sit in `Data.InterestRateLegsInstrument`, so a
  note is the leg and its notional, and a swap the leg and a funding leg.
  Each day is priced from the smile, on caplet models
  (`RatesOption.CapletModels`) or CMS models. A floating coupon not yet fixed
  also takes a one-factor Markov model, such as `RatesOption.LgmModel`.
- **Callable interest rate legs**, `Data.CallableInterestRateLegs`: legs of
  any kind that one side may call, at par or at call prices, such as
  callable notes and cancellable swaps.
  `RatesOption.pvCallableInterestRateLegsQuadrature` and
  `pvCallableInterestRateLegsFiniteDifference` price them on a one-factor
  Markov model, with greeks to match.
  `RatesOption.applyCallableInterestRateLegsCall` applies a call.
- **Cashflows of Bermudan swaptions and callable legs**, at zero volatility:
  `RatesOption.bermudanSwaptionTermRateCashflows`,
  `bermudanSwaptionOisCashflows` and `callableInterestRateLegsCashflows`.
- **A speed optimisation for calibrating base correlations again after
  quotes change**, `CreditCorrelation.BaseCorrelationPool`. It keeps the rows before the first
  changed quote and the loss distributions, and matches
  `calibrateBaseCorrelation`'s results to the last bit.
- **Calls from several threads at once.** qeLib keeps no state shared between
  calls without a lock. The ISDA CDS Standard Model's code has been changed
  for this alone; its results are unchanged, and `licenses/NOTICES.txt` lists
  the changes.

### Changed

- Base correlation calibration solves each row by secant steps inside its
  bracket, in fewer evaluations. Base correlations change within the
  calibration's tolerance.
- The greeks of tranches, and the Jacobian of base correlations, take about a
  quarter less time.
- Beyond the quoted strikes, the smiles of stripped caplet volatilities and of
  the swaption cube are smooth in the volatilities, so that greeks hedge to
  second order.

### Known issues

- Range accruals priced on stripped caplet volatilities
  (`RatesOption.CapletVolatilityCalculatorStripped`) are strongly curved in
  the volatilities quoted at the first and last strikes. Beyond those
  strikes, the smile's tail widens with its slope at the end, so a hedge
  sized by the greeks leaves more of a day's move than on SABR caplet models
  (`RatesOption.CapletSabrPiecewise`), whose smiles have no such tail.

## 0.1.0.post1 - 2026-11-10

The 0.1.0 wheels again, with only their metadata corrected: from PyPI, the
Linux wheel wouldn't install, PyPI having taken the Windows wheel's Python
3.13 requirement for every 0.1.0 file. Both wheels now require Python 3.13 or
later, their tags saying which Python each is for. The library is unchanged,
and still reports version 0.1.0; the files on the GitHub Release were never
affected.

## 0.1.0 - 2026-11-10

The first release of qeLib, a library of quantitative finance analytics for
Excel and Python.

### What a release contains

- The Excel add-in for 64-bit Excel on Windows, with the User Guide, example
  workbooks and the log tool, which turns a log of qeLib calls into a
  workbook.
- Python wheels for Python 3.13 on 64-bit Windows and Python 3.14 on x86-64
  Linux: `pip install qelib`, then `import qeLib`.
- An evaluation license, valid for 90 days. Named developer licenses are
  available on request.

Every function is available as `qeLib.<Service>.<Name>`, in both Excel and
Python.

### Services

- **Data:** dates, tenors, holiday calendars and business-day conventions;
  market data; trades and their legs.
- **Core:** numerics and date arithmetic.
- **IsdaCds:** the ISDA CDS Standard Model, with CDS from a flat spread or
  points upfront.
- **Curves:** discount, projection, survival and recovery curves. CDS, OIS,
  multi-curve and cross-currency stripping, with turns. Swap and CDS
  pricing.
- **CreditCorrelation:** one-factor copulas, recovery models and loss
  distributions for index and bespoke tranches and nth-to-default baskets,
  with base correlations calibrated to tranche quotes or expected losses.
- **RatesOption:** caps, floors, swaptions and CMS, on Black-Scholes
  volatilities or SABR, with SABR calibrated to cap volatilities and
  swaption cubes. Bermudan swaptions, by finite differences or by
  quadrature.
- **CreditOption:** options on CDS indices, with the default-adjusted
  forward, volatility surfaces and implied spread densities.
- **SRT:** significant risk transfer securitisations under the EU CRR: the
  capital before and after a transfer, the risk transfer tests, pool
  selection, tranche thickness and the output floor.
- **FxOption:** foreign exchange options on SABR, Vanna-Volga or SVI smiles
  calibrated to market quotes.
- **Bonds:** government bonds and bills: yields and prices in each market's
  convention, curve pricing, z-spreads, and government curves fitted to
  prices, including Nelson-Siegel-Svensson. Classic term repos and general
  collateral repos on them.

### Across the services

- Greeks by adjoint algorithmic differentiation, projected onto the quotes
  the curves were built from.
- Hedge notionals: the amounts of chosen hedge trades that offset a book's
  greeks.
- Expected cashflows for every instrument.
