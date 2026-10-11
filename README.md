# iDMS Documentation

Official system architecture, integration guides, and user manuals for the **iDMS** (iRODS Data Management System) suite.

## Live Documentation Site
You can read the fully rendered, searchable documentation site online at:
> **https://gitpages.rivm.nl/idms-docs-40becf/**

## Local Development (Building the Docs)
If you want to preview or edit the documentation locally:

1. Clone this repository and navigate into it.

2. Install the documentation dependencies:

   ```bash
   pip install -r requirements.txt
   ```

   or

   ```bash
   conda env create -f envs/idms-docs.yml
   conda activate idms-docs
   ```


3. Run the local live-reload server:
   ```bash
    mkdocs serve
   ```
4. Open http://127.0.0.1:8000 in your browser.
   Any changes (after saving) will be reloaded immediatly.
   
5. merge and push changes to origin/main.
   GitLab-Pages will be rebuild automatically


## convertion of wiki pages to md files

Internal links and images need to be changed:
   
```bash
  find ./docs/ -name "*.md" | xargs grep -o -E "\[.*\]\(.*\)" | wc -l
```

