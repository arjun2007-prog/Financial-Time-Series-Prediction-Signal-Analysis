# Nifty 50 Open-to-Close Returns — Reproducibility Guide

## Purpose
Adds two derived return columns (`simple_return_o2c`, `log_return_o2c`) to a cleaned Nifty 50 daily OHLCV file, with no changes to the original data.

## Input
- **File**: `cleaned_data.csv`
- **Required columns**: `Date, Close, High, Low, Open, Volume`
- **Assumptions**: data is index-level (no adjustment needed), already cleaned (no nulls, no OHLC integrity violations, no zero/negative prices), one row per trading day, sorted or sortable by `Date`.

## Output
- **File**: `cleaned_data_with_returns.csv`
- Same rows and original columns as input, plus 2 new columns appended at the end.

## Environment
```bash
python >= 3.9
pandas
numpy
```
```bash
pip install pandas numpy
```

## Steps

1. **Load and sort**
   ```python
   import pandas as pd
   import numpy as np

   df = pd.read_csv('cleaned_data.csv')
   df['Date'] = pd.to_datetime(df['Date'])
   df = df.sort_values('Date').reset_index(drop=True)
   ```

2. **Compute open-to-close returns**
   ```python
   df['simple_return_o2c'] = (df['Close'] / df['Open']) - 1
   df['log_return_o2c'] = np.log(df['Close'] / df['Open'])
   ```

3. **Validate** (optional but recommended)
   ```python
   assert np.allclose(df['log_return_o2c'], np.log(1 + df['simple_return_o2c']), atol=1e-8)
   assert df[['simple_return_o2c', 'log_return_o2c']].isna().sum().sum() == 0
   ```

4. **Save**
   ```python
   df.to_csv('cleaned_data_with_returns.csv', index=False)
   ```

## Full Script
```python
import pandas as pd
import numpy as np

df = pd.read_csv('cleaned_data.csv')
df['Date'] = pd.to_datetime(df['Date'])
df = df.sort_values('Date').reset_index(drop=True)

df['simple_return_o2c'] = (df['Close'] / df['Open']) - 1
df['log_return_o2c'] = np.log(df['Close'] / df['Open'])

assert np.allclose(df['log_return_o2c'], np.log(1 + df['simple_return_o2c']), atol=1e-8)
assert df[['simple_return_o2c', 'log_return_o2c']].isna().sum().sum() == 0

df.to_csv('cleaned_data_with_returns.csv', index=False)
```

## Verification Checklist
- [ ] Row count unchanged from input (2,670 rows)
- [ ] Original 6 columns unchanged in values and order
- [ ] `simple_return_o2c` and `log_return_o2c` present, no nulls
- [ ] `log_return_o2c ≈ ln(1 + simple_return_o2c)` holds for all rows
- [ ] `Date` column parses correctly and is sorted ascending

## Notes on Reproducibility
- Deterministic: no randomness, no external API calls, no lookback window — every row's return depends only on that row's own `Open` and `Close`.
- Re-running the script on the same input will always produce an identical output file.
- See `DATASHEET.md` for full schema and field-level documentation.
