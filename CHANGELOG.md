# Changelog

Each qeLib release, newest first. While the major version is 0, a minor
release may change interfaces; a patch release only fixes.

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
