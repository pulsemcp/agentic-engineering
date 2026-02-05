# MCP Server Setup

External setup required for each server in `servers.json`. Complete these steps before activating a server.

---

## io.github.awslabs/log-analyzer-cloudwatch

**Server:** `io.github.awslabs/log-analyzer-cloudwatch`
**Repository:** [awslabs/Log-Analyzer-with-MCP](https://github.com/awslabs/Log-Analyzer-with-MCP)

### Prerequisites

- **Python 3.12+** and [uv](https://docs.astral.sh/uv/getting-started/installation/) (provides the `uvx` command)
- An **AWS account** with CloudWatch Logs data

### AWS credentials setup

The server needs AWS credentials with CloudWatch Logs read access. Choose one approach:

1. **AWS CLI profiles** (recommended for local dev):
   - Install the [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
   - Run `aws configure` or `aws configure sso` to set up a named profile
   - Set `AWS_PROFILE` to your profile name

2. **Static credentials** (for CI or environments without the CLI):
   - Create an IAM user or role with `CloudWatchLogsReadOnlyAccess`
   - Set `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, and optionally `AWS_SESSION_TOKEN`

3. **Instance/container roles** (for EC2, ECS, Lambda):
   - Attach a role with `CloudWatchLogsReadOnlyAccess` — no env vars needed

### Required IAM permissions

At minimum, the credentials need these CloudWatch Logs actions:

- `logs:DescribeLogGroups`
- `logs:DescribeLogStreams`
- `logs:GetLogEvents`
- `logs:FilterLogEvents`
- `logs:StartQuery`
- `logs:GetQueryResults`
- `logs:StopQuery`

The managed policy `CloudWatchLogsReadOnlyAccess` covers all of these.

### Environment variables

| Variable | Required | Description |
|---|---|---|
| `AWS_PROFILE` | No | Named AWS CLI profile to use |
| `AWS_DEFAULT_REGION` | No | AWS region (e.g., `us-east-1`). Defaults to profile/CLI default. |
| `AWS_ACCESS_KEY_ID` | No | Access key for static credential auth |
| `AWS_SECRET_ACCESS_KEY` | No | Secret key for static credential auth |
| `AWS_SESSION_TOKEN` | No | Session token for temporary credentials |

### Verify setup

```bash
# Confirm AWS credentials are working and can access CloudWatch Logs
aws logs describe-log-groups --max-items 1
```

If this returns a log group, the server will work. See the [AWS config guide](https://github.com/awslabs/Log-Analyzer-with-MCP/blob/main/docs/aws-config.md) for more details.

---

## io.github.crystaldba/postgres-mcp

**Server:** `io.github.crystaldba/postgres-mcp`
**Repository:** [crystaldba/postgres-mcp](https://github.com/crystaldba/postgres-mcp)

### Prerequisites

- **Python 3.12+** and [uv](https://docs.astral.sh/uv/getting-started/installation/) (provides the `uvx` command)
- A **PostgreSQL** database you want to connect to

### Optional PostgreSQL extensions

For full functionality (index tuning and hypothetical index analysis), install these extensions on the target database:

- **pg_stat_statements** — query execution statistics (`CREATE EXTENSION IF NOT EXISTS pg_stat_statements;`)
- **hypopg** — hypothetical index simulation (`CREATE EXTENSION IF NOT EXISTS hypopg;`)

These are optional; the server works without them but some analysis features will be limited.

### Environment variables

| Variable | Required | Description |
|---|---|---|
| `PG_USER` | Yes | PostgreSQL username |
| `PG_PASSWORD` | Yes | PostgreSQL password |
| `PG_HOST` | Yes | PostgreSQL hostname (e.g., `localhost`) |
| `PG_PORT` | Yes | PostgreSQL port (e.g., `5432`) |
| `PG_DATABASE` | Yes | PostgreSQL database name (e.g., `postgres`) |

These are composed into `DATABASE_URI` as `postgresql://$PG_USER:$PG_PASSWORD@$PG_HOST:$PG_PORT/$PG_DATABASE`.

### Access modes

The `--access-mode` flag controls what the server can do:

- `restricted` (default) — read-only queries with resource limits, safe for production
- `unrestricted` — full read/write access, intended for development

### Verify setup

```bash
# Confirm the database is reachable
psql "postgresql://$PG_USER:$PG_PASSWORD@$PG_HOST:$PG_PORT/$PG_DATABASE" -c "SELECT 1;"
```

If this returns a row, the server will be able to connect.

---

## io.github.awslabs/aws-iac-mcp-server

**Server:** `io.github.awslabs/aws-iac-mcp-server`
**Repository:** [awslabs/mcp](https://github.com/awslabs/mcp) (subfolder `src/aws-iac-mcp-server`)

### Prerequisites

- **Python 3.10+** and [uv](https://docs.astral.sh/uv/getting-started/installation/) (provides the `uvx` command)
- An **AWS account** (for CloudFormation validation and deployment troubleshooting)

### AWS credentials setup

The server needs AWS credentials for CloudFormation and CDK operations. Choose one approach:

1. **AWS CLI profiles** (recommended for local dev):
   - Install the [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
   - Run `aws configure` or `aws configure sso` to set up a named profile
   - Set `AWS_PROFILE` to your profile name

2. **Static credentials** (for CI or environments without the CLI):
   - Create an IAM user or role with appropriate CloudFormation permissions
   - Set `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, and optionally `AWS_SESSION_TOKEN`

3. **Instance/container roles** (for EC2, ECS, Lambda):
   - Attach a role with appropriate CloudFormation permissions — no env vars needed

### Environment variables

| Variable | Required | Description |
|---|---|---|
| `AWS_PROFILE` | No | Named AWS CLI profile to use |
| `AWS_DEFAULT_REGION` | No | AWS region (e.g., `us-east-1`). Defaults to profile/CLI default. |
| `AWS_ACCESS_KEY_ID` | No | Access key for static credential auth |
| `AWS_SECRET_ACCESS_KEY` | No | Secret key for static credential auth |
| `AWS_SESSION_TOKEN` | No | Session token for temporary credentials |
| `FASTMCP_LOG_LEVEL` | No | Logging level (default: `ERROR`) |

### Verify setup

```bash
# Confirm AWS credentials are working
aws sts get-caller-identity
```

If this returns your account info, the server will work.

---

## io.github.xing5/mcp-google-sheets

**Server:** `io.github.xing5/mcp-google-sheets`
**Repository:** [xing5/mcp-google-sheets](https://github.com/xing5/mcp-google-sheets)

### Prerequisites

- **Python 3.10+** and [uv](https://docs.astral.sh/uv/getting-started/installation/) (provides the `uvx` command)
- A **Google Cloud Platform** project

### Google Cloud setup

1. **Create or select a GCP project** at [console.cloud.google.com](https://console.cloud.google.com)

2. **Enable APIs** — go to **APIs & Services > Library** and enable:
   - Google Sheets API
   - Google Drive API

3. **Create a Service Account:**
   - Go to **APIs & Services > Credentials**
   - Click **+ Create Credentials > Service account**
   - Name it (e.g. `mcp-sheets-sa`), click **Create and Continue**
   - Skip the optional role/access steps, click **Done**
   - Click the new service account, go to the **Keys** tab
   - Click **Add Key > Create new key > JSON > Create** — a `.json` key file downloads

4. **Base64-encode the key** (no key file needed on disk at runtime):
   ```bash
   base64 -i /path/to/downloaded-key.json
   ```
   Copy the output — this is your `GOOGLE_SHEETS_CREDENTIALS_CONFIG` value.

5. **Create a shared Drive folder:**
   - Go to [drive.google.com](https://drive.google.com) and create a new folder (e.g. `MCP Sheets`)
   - Right-click the folder > **Share**
   - Paste the service account email (found in GCP console under **IAM & Admin > Service Accounts**, looks like `mcp-sheets-sa@your-project.iam.gserviceaccount.com`)
   - Give it **Editor** access, click **Send**
   - Open the folder — the folder ID is in the URL: `https://drive.google.com/drive/folders/<FOLDER_ID>`

### Environment variables

| Variable | Required | Description |
|---|---|---|
| `GOOGLE_SHEETS_CREDENTIALS_CONFIG` | Yes | Base64-encoded Service Account JSON key |
| `GOOGLE_SHEETS_DRIVE_FOLDER_ID` | Yes | Google Drive folder ID shared with the service account |

### Verify setup

```bash
CREDENTIALS_CONFIG="$(cat <<< "$GOOGLE_SHEETS_CREDENTIALS_CONFIG")" \
DRIVE_FOLDER_ID="$GOOGLE_SHEETS_DRIVE_FOLDER_ID" \
uvx mcp-google-sheets==0.6.0
```

If the server starts without errors, the credentials and folder access are working.
