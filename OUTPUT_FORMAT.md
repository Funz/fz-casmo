# CASMO Output Format

This document describes the typical CASMO5 output format that the fz-casmo plugin expects.

## Burnup Table Format

CASMO5 produces tabular output showing various parameters as a function of burnup. The plugin is designed to parse these tables.

### Example Output Table

A typical CASMO5 output file contains burnup tables like this:

```
 BURNUP     K-INF       M2      B10-ABS    XE135-ABS  ...
  (MWD/KG)             (CM**2)   (1/CM)     (1/CM)

    0.000   1.22534    15.832    0.00123    0.00000
    5.000   1.18765    15.241    0.00118    0.00456
   10.000   1.15234    14.723    0.00114    0.00501
   15.000   1.11956    14.287    0.00110    0.00523
   20.000   1.08895    13.924    0.00106    0.00538
   25.000   1.06021    13.625    0.00102    0.00548
   30.000   1.03310    13.381    0.00099    0.00555
   35.000   1.00742    13.186    0.00096    0.00560
   40.000   0.98301    13.033    0.00093    0.00563
   45.000   0.95972    12.917    0.00090    0.00565
   50.000   0.93741    12.833    0.00088    0.00567
```

### Column Descriptions

- **BURNUP**: Cumulative burnup in MWd/kgU (or MWd/kg)
- **K-INF**: Infinite multiplication factor (neutron economy measure)
- **M2**: Migration area in cm² (also called buckling)
- **B10-ABS**: B-10 absorption rate
- **XE135-ABS**: Xe-135 absorption rate
- Additional columns may include isotope concentrations, cross-sections, etc.

## Default Output Parsing

The plugin's default configuration (`.fz/models/CASMO.json`) extracts the first three columns:

```json
{
    "output": {
        "burnup": "grep -A 1000 'BURNUP' output.txt | grep -E '^[[:space:]]*[0-9]' | awk '{print $1}'",
        "k_inf": "grep -A 1000 'BURNUP' output.txt | grep -E '^[[:space:]]*[0-9]' | awk '{print $2}'",
        "m2": "grep -A 1000 'BURNUP' output.txt | grep -E '^[[:space:]]*[0-9]' | awk '{print $3}'"
    }
}
```

### How the Parsing Works

1. **grep -A 1000 'BURNUP'**: Finds the line containing "BURNUP" and extracts 1000 lines after it
2. **grep -E '^[[:space:]]*[0-9]'**: Filters lines starting with whitespace followed by a digit (data rows)
3. **awk '{print $N}'**: Extracts column N from each data row

## Customizing Output Parsing

You can extract additional columns by modifying `.fz/models/CASMO.json`:

### Example: Extract Additional Columns

```json
{
    "id": "CASMO",
    "varprefix": "$",
    "formulaprefix": "@",
    "delim": "{}",
    "commentline": "*",
    "output": {
        "burnup": "grep -A 1000 'BURNUP' output.txt | grep -E '^[[:space:]]*[0-9]' | awk '{print $1}'",
        "k_inf": "grep -A 1000 'BURNUP' output.txt | grep -E '^[[:space:]]*[0-9]' | awk '{print $2}'",
        "m2": "grep -A 1000 'BURNUP' output.txt | grep -E '^[[:space:]]*[0-9]' | awk '{print $3}'",
        "b10_abs": "grep -A 1000 'BURNUP' output.txt | grep -E '^[[:space:]]*[0-9]' | awk '{print $4}'",
        "xe135_abs": "grep -A 1000 'BURNUP' output.txt | grep -E '^[[:space:]]*[0-9]' | awk '{print $5}'"
    }
}
```

### Example: Extract Specific Values

If you want just the final burnup k-inf:

```json
{
    "output": {
        "final_kinf": "grep -A 1000 'BURNUP' output.txt | grep -E '^[[:space:]]*[0-9]' | tail -1 | awk '{print $2}'"
    }
}
```

### Example: Parse Different Table

CASMO5 may have multiple tables. To parse a different table:

```json
{
    "output": {
        "pin_powers": "grep -A 100 'PIN POWER' output.txt | grep -E '^[[:space:]]*[0-9]' | awk '{print $0}'"
    }
}
```

## Multiple Output Tables

CASMO5 can produce multiple types of output tables:

### 1. Burnup-Dependent Parameters
- k-infinity, migration area, cross-sections
- Isotope concentrations (U-235, U-238, Pu-239, etc.)
- Neutron spectrum indices

### 2. Pin Power Distributions
- Relative pin powers
- Form factors
- Assembly power distributions

### 3. Cross-Section Tables
- Microscopic cross-sections
- Macroscopic cross-sections
- Group-collapsed cross-sections

### 4. Depletion Data
- Heavy metal masses
- Fission product inventories
- Actinide buildup

## Parsing Strategy

For complex outputs, you can:

1. **Use multiple grep patterns**: Target specific table headers
2. **Use sed/awk processing**: Extract structured data
3. **Write Python scripts**: For complex parsing logic

### Example: Python Script for Parsing

```json
{
    "output": {
        "isotopes": "python -c 'import sys; data=open(\"output.txt\").read(); print(data.split(\"ISOTOPE\")[1].split(\"\\n\")[2:10])'"
    }
}
```

## Output Data Types

The fz framework automatically converts output values:

- **Single numbers**: `1.22534` → `1.22534` (float)
- **Arrays**: `[0.0, 5.0, 10.0]` → List of values
- **JSON strings**: Automatically parsed

Example result DataFrame:

```python
   enrichment  burnup                              k_inf                              m2
0         3.0  [0.0, 5.0, 10.0, ...]  [1.225, 1.188, 1.152, ...]  [15.8, 15.2, 14.7, ...]
1         4.0  [0.0, 5.0, 10.0, ...]  [1.314, 1.278, 1.243, ...]  [16.5, 15.9, 15.4, ...]
```

## Troubleshooting Output Parsing

### Issue: No data extracted

**Possible causes:**
1. Output format differs from expected
2. Table header not found
3. Wrong column numbers

**Solutions:**
1. Check `results/*/output.txt` to see actual CASMO5 output
2. Adjust grep pattern to match your output
3. Test parsing commands manually:
   ```bash
   grep -A 1000 'BURNUP' output.txt | grep -E '^[[:space:]]*[0-9]' | head
   ```

### Issue: Wrong column extracted

**Solution:** Count columns in your output (columns are space-separated):
```bash
grep -A 1000 'BURNUP' output.txt | grep -E '^[[:space:]]*[0-9]' | head -1 | awk '{for(i=1;i<=NF;i++) print i, $i}'
```

### Issue: Multiple tables with same header

**Solution:** Use more specific patterns or add context:
```bash
grep -B 2 -A 1000 'BURNUP' output.txt | grep -A 1000 'K-INF' | ...
```

## Examples

See `example_pwr_lattice.py` for a complete working example of running parametric studies and handling the parsed output.

## References

- CASMO5 User Manual (Studsvik)
- "CASMO-5 Development and Applications" (PHYSOR 2006)
- MIT OCW 22.251: Systems Analysis of the Nuclear Fuel Cycle
