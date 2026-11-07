# qeLib

qeLib is a library of quantitative finance analytics, used from Excel or from
Python. This repository is where its releases will be published.

**There is no release yet.** The first version will be 0.1.0.

## What it covers

- Discount, projection, survival and recovery curves, with CDS, OIS,
  multi-curve and cross-currency stripping, and swap pricing.
- CDS through the ISDA CDS Standard Model.
- Copulas, recovery models and loss distributions for credit tranches, and
  options on CDS indices.
- Caps, floors, swaptions and CMS, on flat volatilities or SABR, and Bermudan
  swaptions.
- Foreign exchange options on a smile calibrated to market quotes.
- Government bonds and bills: yields, prices, curves fitted to them, and their
  values and risk.
- Analytics for Significant Risk Transfer securitisations.

Every function is available as `qeLib.<Service>.<Name>`, in both Excel and
Python.

## What each release will contain

- The Excel add-in for 64-bit Excel on Windows.
- Python wheels for Python 3.13 on 64-bit Windows and Python 3.14 on x86-64
  Linux.
- The User Guide.
- Example workbooks.

qeLib's source code is private. Each release will be published here under
qeLib's developer license, with an evaluation license so that anyone can try it.
For commercial licenses, please see contact details below.

## SPI

qeLib uses the SPI runtime libraries, which are under the LGPL. The SPI source
that qeLib builds on is public at
[mortenbaiandersen-cpu/spi](https://github.com/mortenbaiandersen-cpu/spi), on the
branch `qelib`. Each release will name the exact commit it was built from.

## Contact

licensing@qeanalytics.co.uk
