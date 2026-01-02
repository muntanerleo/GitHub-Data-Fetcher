# Data Validation and Processing Pipeline

Complete solution for fetching, validating, and processing JSON files from the owlmaps/map-data repository.

## Overview

This pipeline performs three main operations:
1. **Fetch** - Download recent files from GitHub repository
2. **Validate** - Check data structure and business rules
3. **Process** - Separate valid data into `raw_data` variable, save invalid to `failed/` directory

## Files in This Package

### Core Components
- `validate_and_process.py` - Complete pipeline (fetch → validate → process)
- `demo_validation_pipeline.py` - Demo with local files
- `fetch_recent_data.py` - GitHub data fetcher
- `usage_examples.py` - Usage examples for fetcher

### Validation Components
- `conflict_data_schema.json` - JSON schema definition
- `validate_data_simple.py` - Standalone validator

### Documentation
- `README.md` - GitHub fetcher documentation
- `VALIDATION_PIPELINE.md` - This file

## Quick Start

### Option 1: Complete Pipeline (Fetch + Validate)

```python
from validate_and_process import main

# Run complete pipeline
raw_data, recent_files = main()

# raw_data now contains only validated datasets
# Failed files are in ./failed/ directory
```

### Option 2: Validate Local Files

```python
from demo_validation_pipeline import LocalDataPipeline

pipeline = LocalDataPipeline(failed_dir="/path/to/failed")
files_to_validate = ['/path/to/file1.json', '/path/to/file2.json']

raw_data, passed, failed = pipeline.process_files(files_to_validate)
```

## How It Works

### Step 1: Fetch Recent Files

```python
from fetch_recent_data import GitHubDataFetcher

fetcher = GitHubDataFetcher()
recent_files = fetcher.fetch_and_filter(num_recent=5)

# recent_files is a list of file metadata dictionaries
```

### Step 2: Validate Each File

For each file in `recent_files`:

1. **Download content** from GitHub
2. **Structure validation** - Check required fields and data types
3. **Business rules validation** - Check coordinate ranges, unit counts, etc.

### Step 3: Categorize Results

**PASSED Validation:**
- Data is added to `raw_data` list
- Ready for further processing

**FAILED Validation:**
- Original JSON saved to `failed/` directory
- Error report saved as `{filename}_errors.txt`

## Validation Rules

### Structure Validation (Must Pass)

Required fields:
- `areas` - Array of polygon coordinates
- `areas_ua` - Array of Ukrainian area polygons
- `frontline` - Array of frontline coordinates
- `geos` - Object with `ru` and `ua` incident arrays
- `unit_count` - Object with `ru` and `ua` counts
- `units` - Object with `ru` and `ua` unit arrays

Each field must have the correct type and structure.

### Business Rules Validation (Warnings)

- Coordinate ranges: Longitude [-180, 180], Latitude [-90, 90]
- Unit count consistency
- No duplicate unit IDs

## Output Format

### `raw_data` Variable

```python
raw_data = [
    {
        "areas": [...],
        "areas_ua": [...],
        "frontline": [...],
        "geos": {
            "ru": [...],
            "ua": [...]
        },
        "unit_count": {"ru": 867, "ua": 493},
        "units": {
            "ru": [...],
            "ua": [...]
        }
    },
    # ... more validated datasets
]
```

### `failed/` Directory Structure

```
failed/
├── 20250815.json              # Original failed data
├── 20250815_errors.txt        # Error report
├── 20250820.json              # Another failed file
└── 20250820_errors.txt        # Its error report
```

### Error Report Format

```
VALIDATION FAILURE REPORT
================================================================================
Filename: 20250815.json

ERRORS (3):
--------------------------------------------------------------------------------
1. Missing required field: areas_ua
2. 'geos' must have 'ua' field
3. units.ru must be an array

WARNINGS (2):
--------------------------------------------------------------------------------
1. [INVALID_COORDINATE] Invalid longitude 200 in geos.ru[0]
2. [COUNT_MISMATCH] Russian unit count mismatch: expected 867, found 866
```

## Example Usage

### Example 1: Basic Pipeline

```python
from validate_and_process import DataPipeline
from fetch_recent_data import GitHubDataFetcher

# Fetch files
fetcher = GitHubDataFetcher()
recent_files = fetcher.fetch_and_filter(num_recent=10)

# Validate and process
pipeline = DataPipeline()
raw_data, passed, failed = pipeline.process_files(recent_files)

print(f"Validated {passed} files successfully")
print(f"raw_data contains {len(raw_data)} datasets")
```

### Example 2: Process Specific Files

```python
from demo_validation_pipeline import LocalDataPipeline

# Create pipeline
pipeline = LocalDataPipeline(failed_dir="/home/user/failed_data")

# Validate specific files
files = [
    '/data/20250901.json',
    '/data/20250831.json',
    '/data/20250830.json'
]

raw_data, passed, failed = pipeline.process_files(files)

# Use validated data
for idx, data in enumerate(raw_data):
    print(f"Dataset {idx+1}:")
    print(f"  Russian units: {len(data['units']['ru'])}")
    print(f"  Ukrainian units: {len(data['units']['ua'])}")
```

### Example 3: Custom Validation

```python
from validate_and_process import DataValidator

validator = DataValidator()

# Load your data
import json
with open('myfile.json', 'r') as f:
    data = json.load(f)

# Validate
is_valid, errors, warnings = validator.validate_file(data, 'myfile.json')

if is_valid:
    print("Validation passed!")
    if warnings:
        print(f"  (with {len(warnings)} warnings)")
else:
    print("Validation failed!")
    for error in errors:
        print(f"  - {error}")
```

## Accessing Validated Data

Once you have `raw_data`, you can access the data like this:

```python
# raw_data is a list of dictionaries
first_dataset = raw_data[0]

# Access different data types
areas = first_dataset['areas']                    # List of polygon coordinates
frontline = first_dataset['frontline']            # List of line coordinates
russian_incidents = first_dataset['geos']['ru']   # List of Russian incidents
ukrainian_incidents = first_dataset['geos']['ua'] # List of Ukrainian incidents
russian_units = first_dataset['units']['ru']      # List of Russian unit positions
ukrainian_units = first_dataset['units']['ua']    # List of Ukrainian unit positions

# Example: Count incidents
total_ru_incidents = len(russian_incidents)
total_ua_incidents = len(ukrainian_incidents)
print(f"Total incidents: {total_ru_incidents + total_ua_incidents}")

# Example: Get all incident coordinates
all_coords = [incident['c'] for incident in russian_incidents + ukrainian_incidents]

# Example: Extract unit IDs
ru_unit_ids = [unit[0] for unit in russian_units]
ua_unit_ids = [unit[0] for unit in ukrainian_units]
```

## Pipeline Output Example

```
================================================================================
COMPLETE DATA PIPELINE: FETCH → VALIDATE → PROCESS
================================================================================

================================================================================
STEP 1: FETCHING RECENT FILES FROM GITHUB
================================================================================
GitHub API is accessible
Found 145 JSON files with valid date names
Selected 5 files

================================================================================
STEP 2: VALIDATING AND PROCESSING FILES
================================================================================

[1/5] Processing: 20250901.json
  → Downloading...
  → Validating...
  ✓ PASSED

[2/5] Processing: 20250831.json
  → Downloading...
  → Validating...
  ✓ PASSED (with 2 warnings)

[3/5] Processing: 20250830.json
  → Downloading...
  → Validating...
  ✗ FAILED: 3 errors found
  → Saved to failed directory: 20250830.json
  → Error report: 20250830_errors.txt

[4/5] Processing: 20250829.json
  → Downloading...
  → Validating...
  ✓ PASSED

[5/5] Processing: 20250828.json
  → Downloading...
  → Validating...
  ✓ PASSED

================================================================================
PIPELINE SUMMARY
================================================================================

Total files processed: 5
  ✓ Passed validation: 4 (80.0%)
  ✗ Failed validation: 1 (20.0%)

Failed files saved to: /home/claude/failed/

raw_data variable contains 4 validated datasets
  Type: list of 4 dictionaries
  Each contains the complete JSON data structure

Sample data structure (first file):
  - areas: 11 polygons
  - frontline: 2 segments
  - geos.ru: 17 incidents
  - geos.ua: 20 incidents
  - units.ru: 866 units
  - units.ua: 492 units

================================================================================
✓ ALL FILES VALIDATED SUCCESSFULLY
================================================================================
```

## Error Handling

The pipeline handles various error conditions:

- **Network errors** - Retry with timeout
- **Download failures** - Skip file, continue processing
- **JSON parse errors** - Mark as failed, save error report
- **Validation errors** - Save to failed directory with details
- **Missing files** - Graceful degradation

## Customization

### Change Number of Files

```python
recent_files = fetcher.fetch_and_filter(num_recent=10)  # Get 10 files
```

### Change Failed Directory

```python
pipeline = DataPipeline(failed_dir="/custom/path/failed")
```

### Add Custom Validation Rules

Extend the `DataValidator` class:

```python
class CustomValidator(DataValidator):
    def validate_custom_rules(self, data):
        errors = []
        # Add your custom validation logic
        return errors
```

## Requirements

- Python 3.6+
- `requests` library (for downloading from GitHub)

## Troubleshooting

**Problem:** "Failed to connect to GitHub API"
- **Solution:** Check network connectivity, verify GitHub is accessible

**Problem:** "JSON parse error"
- **Solution:** File may be corrupted, check the file in the failed directory

**Problem:** "No valid JSON files found"
- **Solution:** Verify repository URL and data path are correct

**Problem:** "All files failed validation"
- **Solution:** Check error reports in failed directory for specific issues

## Next Steps

After obtaining `raw_data`, you can:

1. **Analyze trends** - Compare data across dates
2. **Extract insights** - Calculate statistics, identify patterns
3. **Visualize data** - Create maps, charts, dashboards
4. **Export data** - Convert to other formats (CSV, GeoJSON, etc.)
5. **Build reports** - Generate automated reports

## Support

For issues or questions:
- Check error reports in the `failed/` directory
- Review validation logs for specific error messages
- Consult the JSON schema in `conflict_data_schema.json`