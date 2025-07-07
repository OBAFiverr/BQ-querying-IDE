# Querying BQ Through the IDE - A Guide

This document provides comprehensive instructions for setting up all that is needed to run queries via your IDE.

## Prerequisites

Before running this project, ensure you have the following installed:

- **Python 3.9+**
- **Poetry** (Python dependency management)
- **Google Cloud SDK** (gcloud)
- **FCTL** Fiverr Configuration Tool to manage the configuration files for kubernetes services
- **JFrog and github tokens** https://fiverrdata.atlassian.net/wiki/spaces/k1/pages/4196892703/How+to+Setup+JFrog+Github+Tokens

## Project Structure

```
pg_app_asp_multiplier/
├── config/
├── train/
│   └── example_query.py   # Example BigQuery script
├── pyproject.toml         # Poetry dependencies
├── poetry.lock           # Locked dependencies
└── README.md             # This file
```

## Setup Instructions

### 0. Install FCTL (Fiverr Configuration Tool)

FCTL is required to generate configuration files from YAML templates. Install it using:
`
rm -rf ~/.fctl
rm -rf /Volumes/fiverr_dev/fctl

cd /Volumes/fiverr_dev
git clone git@github.com:fiverr/fctl.git
cd fctl
pyenv local 3.9
scripts/install.sh
`
After running the script run:
fctl env -o kube -t all 

and close the terminal.


### 1. Install Dependencies with Poetry

Navigate to the project directory and install dependencies:

```bash
cd models/pg_app_asp_multiplier
poetry install
```

This installs all required packages including:
- `fiverr-ds-bq-utils` - BigQuery utilities
- `fiverr-fconfig` - Configuration management
- `google-cloud-bigquery` - BigQuery client
- Other ML and data science dependencies

### 2. Configure FCTL (Fiverr Configuration Tool)

FCTL is required to generate the configuration files. The `config.py` file is automatically generated from `config.yml` using FCTL.

**Note**: FCTL generates the `.file` configuration that's necessary for the project to work properly.

### 3. Set Up Google Cloud Authentication

#### 3.1 Check Current Authentication

```bash
gcloud auth list
gcloud config get-value project
```

#### 3.2 Set Up Service Account (Jfrog Credentials)

To access BigQuery, you need the BQ library to automatically authenticate through generating the bq_key.json file.

This happens automatically when you run the query here:
```
bq_handler = BqHandler(
        **conf.bq.settings.to_dict()
    )
```


#### 3.3 Verify Project Access

Check which projects you have access to:
```bash
gcloud projects list
```

**Important**: Ensure you have access to the project specified in your configuration:
- For dev: `fiverr-ds-dev`
- For prod: `fiverr-ds`

### 4. Configure Python Path

The most critical step is setting up the Python path correctly. This project uses relative imports that require the current directory to be in the Python path.

Example of a python path:
```bash
export PYTHONPATH=/Users/omer.benami3/Library/Caches/pypoetry/virtualenvs/pg-app-asp-multiplier-0gqju35p-py3.9/bin/python
```


#### Option A: Set PYTHONPATH Environment Variable (Recommended)
This will only work if you are in the same directory as the project:

```bash
export PYTHONPATH=.
```

#### Option B: Use PYTHONPATH with Poetry Run

```bash
PYTHONPATH=. poetry run python train/example_query.py
```

#### Option C: Configure in IDE/Editor

If using an IDE like PyCharm or VS Code:
1. Set the project root as the working directory
2. Add the project root to Python path in your IDE settings
3. Ensure the Python interpreter is set to the Poetry virtual environment

## Running the Script

### Method 1: Using Poetry with PYTHONPATH (Recommended)

```bash
cd models/pg_app_asp_multiplier
export PYTHONPATH=.
poetry run python train/example_query.py
```

### Method 2: Activate Virtual Environment First

```bash
cd models/pg_app_asp_multiplier
poetry shell
export PYTHONPATH=.
python train/example_query.py
```

### Method 3: Direct Python Execution

```bash
cd models/pg_app_asp_multiplier
PYTHONPATH=. python train/example_query.py
```

## Configuration Details

### BigQuery Configuration

The project uses a configuration system that automatically handles BigQuery settings:

```python
from config import conf

# This automatically loads the correct settings from config.yml
bq_handler = BqHandler(**conf.bq.settings.to_dict())
```

The configuration includes:
- **billing_project**: The project to bill BigQuery queries to
- **project**: The project containing the data
- **gcs_bucket**: Google Cloud Storage bucket for temporary files
- **read_tmp_dataset_name**: Dataset for temporary tables
- **service_account**: Service account credentials

### Environment-Specific Configuration

The `config.yml` file contains different configurations for different environments:

- **dev**: Uses `fiverr-ds-dev` project
- **prod**: Uses `fiverr-ds` project
- **local**: Empty configuration for local development

## Troubleshooting

### Common Issues and Solutions

#### 1. ModuleNotFoundError: No module named 'config'

**Cause**: Python can't find the local modules due to incorrect Python path.

**Solution**: Set PYTHONPATH to include the current directory:
```bash
export PYTHONPATH=.
```

#### 2. 403 Forbidden Error on BigQuery

**Cause**: Insufficient permissions for the specified project.

**Solutions**:
- Check your service account permissions
- Verify you have access to the project in `gcloud projects list`
- Switch to a project you have access to
- Contact your team lead for proper permissions

#### 3. FCTL Configuration Issues

**Cause**: Missing or incorrect FCTL setup.

**Solution**: Ensure FCTL is properly installed and configured to generate the `.file` configuration.

#### 4. Poetry Virtual Environment Issues

**Cause**: Virtual environment not activated or dependencies not installed.

**Solution**:
```bash
poetry install
poetry shell
```

## Key Learning Points

1. **Poetry Management**: Always use `poetry install` to install dependencies and `poetry run` to execute scripts.

2. **Python Path**: The most critical aspect is setting `PYTHONPATH=.` to allow Python to find local modules.

3. **Authentication**: Jfrog and github tokens are essential to access the BQ library.

4. **Configuration**: FCTL generates the necessary configuration files that the project depends on.

5. **Project Access**: Ensure you have the correct permissions for the BigQuery project you're trying to access.

## Example Script Structure

Here's how a typical script should be structured:

```python
from fiverr_ds_bq_utils.bq_handler import BqHandler
from config import conf

def main():
    # Use configuration from config.yml
    bq_handler = BqHandler(**conf.bq.settings.to_dict())
    
    # Your BigQuery operations here
    df = bq_handler.get_data(
        query="SELECT * FROM `project.dataset.table` LIMIT 10",
        format='PARQUET'
    )
    print(df.head())

if __name__ == '__main__':
    main()
```

## Summary

This setup process ensures:
- ✅ Proper dependency management with Poetry
- ✅ Correct Python path configuration
- ✅ BigQuery authentication via J4 credentials
- ✅ Configuration management via FCTL
- ✅ Environment-specific settings
- ✅ Proper module imports and execution

Following these steps will allow you to successfully run BigQuery scripts within this project structure.
