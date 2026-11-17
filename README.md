# Export site hierarchies to Excel

Python 3.9+ script that retrieves Cisco Catalyst Center sites using
`GET /dna/intent/api/v1/sites` and exports them to an Excel workbook.
Although the script's docstring and some error messages still reference
"Get Site V2", the configured endpoint is `/dna/intent/api/v1/sites`.

## Run

Fill in `.env` next to the script:

```dotenv
CATALYST_CENTER_URL="https://catalyst.example.com"
CATALYST_CENTER_USERNAME="your-username"
CATALYST_CENTER_PASSWORD="your-password"
```

The script loads this file automatically without replacing existing environment
variables. `.env` is excluded from Git; do not share it or commit credentials.

```bash
python3 -m pip install -r requirements.txt
python3 export_site_hierarchies.py --output site_hierarchies.xlsx
```

`--url` and `--username` override the configured values. If the password is blank,
the script prompts for it without echoing it. For unattended runs,
provide `CATALYST_CENTER_URL`, `CATALYST_CENTER_USERNAME`, and
`CATALYST_CENTER_PASSWORD` through your environment or secret manager.
Alternatively, supply an existing token as `CATALYST_CENTER_TOKEN` (no username
or password is then required). Credentials are not stored in the workbook.

Username/password authentication uses `POST /dna/system/api/v1/auth/token`.
Site requests send the resulting token in the `X-Auth-Token` header.

### Command-line options

| Option | Behavior |
| --- | --- |
| `--url` | Controller HTTPS origin, such as `https://catalyst.example.com`; no API path or embedded credentials |
| `--username` | Username used when no token is supplied |
| `--output` | Destination `.xlsx` file; defaults to `site_hierarchies.xlsx` in the current working directory |


**TLS warning:** The current script disables certificate verification for both
authentication and site retrieval, and suppresses insecure-request warnings.
HTTPS is required, but the server's certificate is not validated. This exposes
credentials and responses to interception; certificate verification should be
restored before production use.

## Output

The `Site Hierarchies` worksheet contains one row per returned site, with these
seven column headers:

| Column | Current value |
| --- | --- |
| Site Hierarchy | `nameHierarchy`, e.g. `Global/USA/CA/Building/Floor` |
| Site Type | Top-level `type`, blank if absent |
| ipv4GlobalPool | Blank; not currently retrieved or written |
| IP Sub pool name | Blank; not currently retrieved or written |
| ipv4PrefixLength | Blank; not currently retrieved or written |
| ipv4DhcpServers | Blank; not currently retrieved or written |
| ipv4DnsServers | Blank; not currently retrieved or written |

Only the first two columns are populated. The script does not currently request
IP-pool or sub-pool details. Site IDs are validated during retrieval but are not
exported. The header row is frozen and an autofilter is enabled.

No hierarchy, ID, or type filter is applied. Pagination starts at offset 1,
requests up to 500 sites per call, and continues until an empty page is returned.
The offset increases by the number of sites returned, including after a short
page. Each site must have a nonblank string `id` and `nameHierarchy`.
Repeated IDs or malformed results cause an explicit failure instead of silently
producing an incomplete export. Requests use a 10-second connection timeout and
a 60-second read timeout. All sites are retrieved before writing the file.
An empty result produces a workbook containing headers only. An existing output
file is overwritten on a successful export; its parent directory must exist.
Site values are written as text, not Excel formulas. A worksheet supports at most
1,048,575 site rows plus the header. Handled HTTP, validation, and filesystem
errors print `Export failed: ...` and return exit code 1; success returns 0.


