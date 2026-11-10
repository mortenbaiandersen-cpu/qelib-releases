# Changelog

Each qeLib release, newest first. While the major version is 0, a minor
release may change interfaces; a patch release only fixes.

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
