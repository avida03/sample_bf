# Blossom Reports

Blossom Reports loads Chilean and Peruvian shipment data, stores the master CSV in
Cloudflare R2, synchronizes shipment rows with Cloudflare D1, and executes the
weekly reporting notebooks to generate HTML reports.

## Setup

Python 3.11 or newer is recommended. Create and activate a virtual environment,
then install the project dependencies:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python3 -m pip install -r requirements.txt
```

On Windows PowerShell, activate the environment with:

```powershell
.venv\Scripts\Activate.ps1
```

## Environment variables

Create a local `.env` file in the project root. Do not commit this file. The
application reads these variables:

```dotenv
# Cloudflare R2
ACCOUNT_ID=
ACCESS_KEY=
SECRET_KEY=
BUCKET_NAME=
CSV_KEY=

# Cloudflare D1
db_id=
db_api_token=

# Update email
sender_email=
sender_password=
```

`config.py` loads the master CSV from Cloudflare R2 when it is imported, so valid
R2 credentials and network access are required for the normal workflows.

## Main workflows

### Update source data and reports

Place incoming `.csv`, `.xls`, or `.xlsx` files in `Sources/`, then run:

```bash
python base_update_Wrapper.py
```

The wrapper identifies Chile and Peru source files, updates R2 and D1, moves the
processed files to `Sources/Archive/`, executes the notebooks, and asks whether
the generated reports are ready to publish. Review the generated changes before
approving its Git commit and push step.

### Execute all notebooks

```bash
python run_notebooks.py
```

This executes every `.ipynb` file in the current directory, updates each notebook
in place, and creates its corresponding HTML file. To send the update email after
successful execution, run:

```bash
python run_notebooks.py --send_email y
```

To execute only one notebook from Python:

```python
from run_notebooks import run_and_save_notebooks

run_and_save_notebooks(["US_Weekly.ipynb"])
```

### Load a source file directly

```bash
python add_to_base.py <source-file> <base-folder-or-cf> <Chile-or-Peru>
```

Use `cf` as the base argument to update the Cloudflare-backed master data. A local
folder argument updates that folder's `Master.csv` instead.

### Send the update email

```bash
python send_update.py
```

## Project files

- `config.py` contains report settings and the Cloudflare R2 client.
- `database_query.py` sends parameterized SQL queries to Cloudflare D1.
- `add_to_base.py` validates, transforms, and loads Chile and Peru source data.
- `base_update_Wrapper.py` coordinates the complete update workflow.
- `run_notebooks.py` executes notebooks and exports HTML reports.
- `send_update.py` sends the site-update notification email.
- `Peru_blossomapp.py` contains Peru-specific transformations and mappings.

## Reports

- `Weekly.ipynb` — Latin America and Polar Fruit weekly import summary.
- `US_Weekly.ipynb` — United States import summary.
- `Us_Grapes.ipynb` — United States grape imports.
- `Us_Sf.ipynb` — United States stone-fruit imports.

Each notebook has a corresponding generated HTML report. `index.html` is the site
landing page and links to the report pages.
