# GitHub Data Fetcher - owlmaps/map-data

A Python tool to programmatically fetch and filter the most recent JSON files from the [owlmaps/map-data](https://github.com/owlmaps/map-data) repository based on their date-based filenames.

## Overview

This tool queries the GitHub repository, identifies JSON files with date-based naming (YYYYMMDD.json), and retrieves the most recent files for further processing.

## Features

- **URL Accessibility Check** - Verifies GitHub API is reachable before proceeding
- **Automatic Date Parsing** - Extracts and validates dates from filenames
- **Smart Filtering** - Sorts files by date (newest first)
- **Flexible Selection** - Choose how many recent files to retrieve
- **Rich Metadata** - Returns file size, download URLs, and git hashes
- **Error Handling** - Comprehensive error handling for network issues
- **Summary Statistics** - Provides overview of repository contents

## Files

- `fetch_recent_data.py` - Main fetcher implementation
- `usage_examples.py` - Examples showing various use cases
- `conflict_data_schema.json` - JSON schema for data validation
- `validate_data_simple.py` - Validation script for downloaded data

## Quick Start

### Basic Usage

```python
from fetch_recent_data import GitHubDataFetcher

# Create fetcher instance
fetcher = GitHubDataFetcher()

# Fetch 5 most recent files
recent_files = fetcher.fetch_and_filter(num_recent=5)

# Access the data
for file in recent_files:
    print(f"{file['filename']} - {file['date'].strftime('%Y-%m-%d')}")
    print(f"  Download: {file['download_url']}")
```

### Output Structure

The `recent_files` variable contains a list of dictionaries:

```python
[
    {
        'filename': '20250901.json',
        'date': datetime.datetime(2025, 9, 1, 0, 0),
        'date_str': '20250901',
        'download_url': 'https://raw.githubusercontent.com/owlmaps/map-data/master/data/20250901.json',
        'size': 458392,  # bytes
        'sha': 'abc123def456...'  # git hash
    },
    # ... more files
]
```

## Workflow Steps

The fetcher follows a systematic 6-step process:

### Step 1: Check Accessibility
```
✓ GitHub API is accessible (Status: 200)
  URL: https://api.github.com/repos/owlmaps/map-data/contents/data
```

### Step 2: Fetch Directory Contents
```
✓ Successfully fetched directory contents
  Total items found: 150
```

### Step 3: Filter JSON Files
```
✓ Found 145 JSON files with valid date names
  ⚠ Skipped 5 files with invalid dates
```

### Step 4: Sort by Date
```
✓ Sorted 145 files (newest to oldest)
```

### Step 5: Select Recent Files
```
✓ Selected 5 files:

#    Date         Filename              Size (KB)
--------------------------------------------------------------------------------
1    2025-09-01   20250901.json             447.7
2    2025-08-31   20250831.json             446.9
3    2025-08-30   20250830.json             445.2
4    2025-08-29   20250829.json             444.8
5    2025-08-28   20250828.json             443.5
```

### Step 6: Display Summary
```
Total JSON files: 145
Date range: 2025-03-01 to 2025-09-01
Days covered: 184
Total size: 63.42 MB
Average file size: 0.44 MB
```

## Advanced Usage

### Fetch More Files

```python
fetcher = GitHubDataFetcher()
recent_files = fetcher.fetch_and_filter(num_recent=10)
```

### Download Files Locally

```python
from fetch_recent_data import GitHubDataFetcher, download_file
import os

fetcher = GitHubDataFetcher()
recent_files = fetcher.fetch_and_filter(num_recent=3)

# Create download directory
os.makedirs("/path/to/downloads", exist_ok=True)

# Download each file
for file_info in recent_files:
    save_path = f"/path/to/downloads/{file_info['filename']}"
    download_file(file_info, save_path)
```

### Filter by Date Range

```python
from datetime import datetime

fetcher = GitHubDataFetcher()

# Get all files first
files_data = fetcher.fetch_directory_contents()
json_files = fetcher.filter_and_parse_files(files_data)
sorted_files = fetcher.sort_by_date(json_files)

# Filter files from last 30 days
cutoff_date = datetime(2025, 8, 1)
recent_files = [f for f in sorted_files if f['date'] >= cutoff_date]
```

### Process File Contents

```python
import requests
import json

fetcher = GitHubDataFetcher()
recent_files = fetcher.fetch_and_filter(num_recent=3)

for file_info in recent_files:
    # Download and parse JSON
    response = requests.get(file_info['download_url'])
    data = response.json()
    
    # Process the data
    print(f"{file_info['filename']}:")
    print(f"  Areas: {len(data.get('areas', []))}")
    print(f"  Russian units: {len(data.get('units', {}).get('ru', []))}")
    print(f"  Ukrainian units: {len(data.get('units', {}).get('ua', []))}")
```

## API Reference

### GitHubDataFetcher Class

#### `__init__(repo_owner, repo_name, data_path)`
Initialize the fetcher with repository information.

**Parameters:**
- `repo_owner` (str): GitHub repository owner (default: "owlmaps")
- `repo_name` (str): Repository name (default: "map-data")
- `data_path` (str): Path to data directory (default: "data")

#### `check_accessibility() -> bool`
Check if the GitHub API is accessible.

**Returns:** True if accessible, False otherwise

#### `fetch_directory_contents() -> Optional[List[Dict]]`
Fetch directory contents from GitHub API.

**Returns:** List of file information or None if failed

#### `filter_and_parse_files(files_data) -> List[Dict]`
Filter JSON files with date pattern and parse dates.

**Parameters:**
- `files_data` (List[Dict]): Raw file data from GitHub API

**Returns:** List of parsed file information

#### `sort_by_date(json_files, descending=True) -> List[Dict]`
Sort files by date.

**Parameters:**
- `json_files` (List[Dict]): Files to sort
- `descending` (bool): Sort order (default: True for newest first)

**Returns:** Sorted list of files

#### `get_recent_files(sorted_files, n=5) -> List[Dict]`
Get N most recent files.

**Parameters:**
- `sorted_files` (List[Dict]): Pre-sorted files
- `n` (int): Number of files to retrieve (default: 5)

**Returns:** List of N most recent files

#### `fetch_and_filter(num_recent=5) -> Optional[List[Dict]]`
Main method to fetch and filter recent files (runs all steps).

**Parameters:**
- `num_recent` (int): Number of recent files to return (default: 5)

**Returns:** List of most recent files or None if failed

### Helper Functions

#### `download_file(file_info, save_path) -> bool`
Download a file from GitHub.

**Parameters:**
- `file_info` (Dict): File information with download_url
- `save_path` (str): Local path to save the file

**Returns:** True if successful, False otherwise

## Error Handling

The fetcher includes comprehensive error handling:

- **Connection Errors** - Handles network unavailability
- **Timeout Errors** - 10-second timeout for API requests, 30-second for downloads
- **HTTP Errors** - Checks response status codes
- **JSON Parse Errors** - Validates JSON responses
- **Invalid Date Formats** - Skips files with malformed dates

## Data Validation

After fetching files, you can validate them using the included schema:

```python
from validate_data_simple import validate_structure, validate_business_rules
import json

# Load downloaded file
with open('20250901.json', 'r') as f:
    data = json.load(f)

# Validate structure
errors = validate_structure(data)
if errors:
    print("Structure errors:", errors)

# Validate business rules
warnings = validate_business_rules(data)
if warnings:
    print("Warnings:", warnings)
```

## Requirements

- Python 3.6+
- `requests` library

Install requirements:
```bash
pip install requests
```

## License

This tool is provided as-is for working with the public owlmaps/map-data repository.

## Contributing

Feel free to extend or modify this tool for your specific needs.