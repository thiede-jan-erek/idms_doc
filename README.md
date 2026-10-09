# iDMS Documentation

Official system architecture, integration guides, and user manuals for the **iDMS** (integrated Data Management System) suite.

## About the Ecosystem
iDMS is modularized into dedicated Python packages:
- **`idms_common_constants`**: Shared constants and AVU-names.
- **`idms_common_irods`**: iRODS integration and session handling.
- **`idms_persistence`**: Database models and REST-API for project-, process- and processgroup-definitions (PostgreSQL).
- **`idms_services`**: Core backend services providing i.g. data-management, job-engine, etc. .
- **`idms_web`**: Web application interface

## Live Documentation Site
You can read the fully rendered, searchable documentation site online at:
> **[to be done]**

## Local Development (Building the Docs)
If you want to preview or edit the documentation locally:

1. Clone this repository and navigate into it.
2. Install the documentation dependencies:
   ```bash
   pip install -r requirements.txt
   ```
3. Run the local live-reload server:
   ```bash
    mkdocs serve
   ```
4. Open http://127.0.0.1:8000 in your browser.

## License

This project is licensed under the European Union Public Licence (EUPL v1.2) - see the LICENSE file for details. 
