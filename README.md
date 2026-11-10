# qeLib

qeLib is a library of quantitative finance analytics, used from Excel or from
Python. This repository is where its releases are published.

## What it covers

- Discount, projection, survival and recovery curves, with CDS, OIS,
  multi-curve and cross-currency stripping, and swap pricing.
- CDS through the ISDA CDS Standard Model.
- Copulas, recovery models and loss distributions for credit tranches, with
  base correlations calibrated to tranche quotes or expected losses, and
  options on CDS indices.
- Caps, floors, swaptions and CMS, on flat volatilities or SABR, and Bermudan
  swaptions.
- Foreign exchange options on a smile calibrated to market quotes.
- Government bonds and bills: yields, prices, curves fitted to them, and their
  values and risk. Classic term repos and general collateral repos on them.
- Analytics for Significant Risk Transfer securitisations.

Every function is available as `qeLib.<Service>.<Name>`, in both Excel and
Python.

## What each release contains

- The Excel add-in for 64-bit Excel on Windows, with the log tool, which
  turns a log of qeLib calls into a workbook.
- Python wheels for Python 3.13 on 64-bit Windows and Python 3.14 on x86-64
  Linux.
- The User Guide.
- Example workbooks.

The Excel add-in is a zip file attached to each release on the
[Releases](https://github.com/mortenbaiandersen-cpu/qelib-releases/releases)
page. The Python wheels are on PyPI:

    pip install qelib

and then `import qeLib`.

qeLib's source code is private. Each release is published here under qeLib's
developer license, with an evaluation license so that anyone can try it. For
commercial licenses, please see contact details below.

## SPI

qeLib uses the runtime libraries of SPI
([PeterTaylor2/spi](https://github.com/PeterTaylor2/spi)), which are under the
LGPL, unmodified and as separate shared libraries that you may replace. Each
release names the SPI commit its libraries were built from, and publishes that
commit's source beside its other files, as `spi-source-COMMIT.zip`.

## Contact

licensing@qeanalytics.co.uk
