# Migration Guide

This guide helps users migrating from the legacy Funz CASMO plugin to fz-casmo.

## Overview

The fz-casmo plugin is a complete rewrite for the new fz framework (Python-based), replacing the older Funz plugin (Java-based). While the core concepts remain the same, there are several changes in syntax and usage.

## Key Changes

### 1. Framework

**Old (Funz):**
- Java-based framework
- Required Java Runtime Environment
- Used Funz.jar for execution

**New (fz):**
- Python-based framework
- Requires Python 3.7+
- Uses pip for installation
- More flexible and easier to integrate with Python workflows

### 2. Variable Syntax

**Old (Funz):**
```
FUE 1 10.42/?[enrichment]
```

**New (fz):**
```
FUE 1 10.42/$enrichment
```

- Changed from `?[variable]` to `$variable`
- More Unix-shell-like syntax
- Easier to read and write

### 3. Formula Syntax

**Old (Funz):**
```
c Formula: #{calculation}
```

**New (fz):**
```
* Formula: @{calculation}
```

- Changed from `#{...}` to `@{...}`
- Changed comment character from `c` to `*` (CASMO standard)
- Python or R expressions supported

### 4. Running Calculations

**Old (Funz):**
```bash
java -jar Funz.jar Run \
  -m CASMO \
  -if input.cas \
  -iv enrichment=[3.0,3.5,4.0] \
  -oe burnup,k_inf
```

**New (fz):**
```python
import fz

results = fz.fzr(
    "input.cas",
    {"enrichment": [3.0, 3.5, 4.0]},
    "CASMO",
    calculators="Localhost_CASMO",
    results_dir="results"
)
```

Or via CLI:
```bash
fzr input.cas \
  --model CASMO \
  --variables '{"enrichment": [3.0, 3.5, 4.0]}' \
  --calculator Localhost_CASMO \
  --results results/
```

### 5. Configuration Files

**Old (Funz):**
- XML configuration files
- Stored in Funz installation directory
- Complex structure

**New (fz):**
- JSON configuration files
- Stored in `.fz/` directory (project-level)
- Simple, readable structure

**Example - Old XML:**
```xml
<Model name="CASMO">
  <Input>
    <Variable name="enrichment" syntax="?[enrichment]"/>
  </Input>
  <Output>
    <Variable name="k_inf" shell="grep..."/>
  </Output>
</Model>
```

**Example - New JSON:**
```json
{
    "id": "CASMO",
    "varprefix": "$",
    "output": {
        "k_inf": "grep..."
    }
}
```

### 6. Output Handling

**Old (Funz):**
- Output parsed to CSV
- Limited DataFrame manipulation
- Separate tools for visualization

**New (fz):**
- Output returned as pandas DataFrame
- Full Python data manipulation capabilities
- Easy integration with matplotlib, seaborn, etc.

```python
import matplotlib.pyplot as plt

results = fz.fzr(...)
plt.plot(results['enrichment'], results['k_inf'])
plt.show()
```

### 7. Parallel Execution

**Old (Funz):**
```bash
java -jar Funz.jar Run -m CASMO -if input.cas \
  -iv enrichment=[...] \
  -rc "calculator1,calculator2,calculator3"
```

**New (fz):**
```python
results = fz.fzr(
    "input.cas",
    {"enrichment": [...]},
    "CASMO",
    calculators=[
        "Localhost_CASMO",
        "Localhost_CASMO",
        "Localhost_CASMO"
    ]
)
```

- More explicit and clear
- Better load balancing
- Easier to configure

### 8. Caching

**Old (Funz):**
- Built-in cache management
- Required manual cache clearing

**New (fz):**
- Cache as a calculator type
- More flexible cache strategies

```python
results = fz.fzr(
    "input.cas",
    {"enrichment": [...]},
    "CASMO",
    calculators=[
        "cache://previous_results",  # Check cache first
        "Localhost_CASMO"             # Run if not cached
    ]
)
```

## Migration Steps

### Step 1: Install fz Framework

```bash
pip install funz-fz
```

### Step 2: Clone fz-casmo Plugin

```bash
git clone https://github.com/Funz/fz-casmo.git
cd fz-casmo
```

### Step 3: Convert Input Files

1. Replace `?[variable]` with `$variable`
2. Replace `#{formula}` with `@{formula}`
3. Update comment syntax if needed (use `*` for CASMO)

**Example:**

Old:
```
FUE 1 10.42/?[enrichment]
c Calculate: #{?[temperature] + 273.15}
```

New:
```
FUE 1 10.42/$enrichment
* Calculate: @{$temperature + 273.15}
```

### Step 4: Convert Scripts

**Old Bash Script:**
```bash
java -jar Funz.jar Run \
  -m CASMO \
  -if pwr_lattice.cas \
  -iv enrichment=[3.0,3.5,4.0,4.5] \
  -oe k_inf,burnup \
  -od results/
```

**New Python Script:**
```python
import fz

results = fz.fzr(
    input_path="pwr_lattice.cas",
    input_variables={"enrichment": [3.0, 3.5, 4.0, 4.5]},
    model="CASMO",
    calculators="Localhost_CASMO",
    results_dir="results"
)

print(results[['enrichment', 'k_inf', 'burnup']])
```

### Step 5: Update Calculator Configuration

**Old (Funz XML):**
```xml
<Calculator name="local">
  <Code>
    <Binary>$CASMO_PATH/casmo5</Binary>
  </Code>
</Calculator>
```

**New (fz JSON):**

`.fz/calculators/Localhost_CASMO.json`:
```json
{
    "uri": "sh://",
    "models": {
        "CASMO": "bash .fz/calculators/CASMO.sh"
    }
}
```

Set `CASMO_PATH` environment variable:
```bash
export CASMO_PATH="/path/to/casmo5"
```

## Benefits of Migration

1. **Modern Python Ecosystem**: Integration with NumPy, pandas, matplotlib, scikit-learn
2. **Simpler Installation**: pip-based, no Java dependencies
3. **Better Documentation**: Clear examples and guides
4. **Active Development**: Regular updates and improvements
5. **Flexible Configuration**: Project-level configuration files
6. **Interactive Usage**: Easy to use in Jupyter notebooks
7. **Better Error Handling**: Clear error messages and debugging

## Common Issues

### Issue: "CASMO_PATH not set"
**Solution**: Export the environment variable:
```bash
export CASMO_PATH="/opt/studsvik/casmo5"
```

### Issue: Variables not substituted
**Solution**: Check that you're using `$variable` instead of `?[variable]`

### Issue: Output not parsed
**Solution**: Check the output format and adjust parsing commands in `.fz/models/CASMO.json`

## Getting Help

- Read the [README](README.md) for complete documentation
- Check the [QUICKSTART](QUICKSTART.md) guide
- See example scripts in the repository
- Open an issue on GitHub: https://github.com/Funz/fz-casmo/issues

## Compatibility Notes

- fz-casmo works with CASMO5 and CASMO5-5
- Output parsing may need adjustment for different CASMO versions
- Test your specific use case after migration

## Feedback

If you encounter issues during migration or have suggestions for improving this guide, please open an issue or submit a pull request!
