# linear2fizzy

Migrate Linear issues to Fizzy cards with full metadata, labels, and comment history.

## Features

- **Issue Migration**: Transfer titles, descriptions, and metadata from Linear to Fizzy
- **Label Conversion**: Automatically creates Fizzy tags from Linear labels
- **Comment Preservation**: Maintains all comments with author attribution and timestamps
- **Markdown Support**: Descriptions are converted to HTML (with cmark or pandoc)
- **Flexible Filtering**: Filter by team, project, state, or labels
- **Safety Options**: Dry-run mode for previewing changes before execution
- **Reference Tracking**: Cards include links back to original Linear issues

## Installation

Download and make executable:

```bash
curl -O https://raw.githubusercontent.com/robzolkos/linear2fizzy/master/linear2fizzy
chmod +x linear2fizzy
mv linear2fizzy /usr/local/bin/
```

Or clone the repository:

```bash
git clone https://github.com/robzolkos/linear2fizzy.git
cd linear2fizzy
chmod +x linear2fizzy
```

## Requirements

- **Fizzy CLI** (`fizzy`) - [Installation](https://github.com/robzolkos/fizzy-cli/releases)
- **jq** - JSON processing utility
- **curl** - HTTP client (usually pre-installed)
- **Linear API Key** - Personal API key from Linear

### Optional

- **cmark** or **pandoc** - For better markdown-to-HTML conversion

## Configuration

### Fizzy CLI Setup

1. Download from [fizzy-cli releases](https://github.com/robzolkos/fizzy-cli/releases)
2. Verify installation: `fizzy --version`
3. Authenticate with your token:

```bash
fizzy auth login YOUR_TOKEN
```

Get your token from the Fizzy web app profile page. Verify authentication:

```bash
fizzy identity show
```

Optionally set a default account:

```bash
export FIZZY_ACCOUNT="YOUR_ACCOUNT_ID"
```

### Linear API Key

Get your API key from [Linear Settings → API](https://linear.app/settings/api):

1. Click **Create key**
2. Give it a name (e.g., "linear2fizzy")
3. Copy the key (starts with `lin_api_...`)

Set the key via environment variable:

```bash
export LINEAR_API_KEY=lin_api_your_key_here
```

Or copy the sample env file and add your key:

```bash
cp .env.sample .env
# Edit .env with your actual key
```

## Usage

```
linear2fizzy --board BOARD_ID [OPTIONS]
```

### Required

| Option | Description |
|--------|-------------|
| `--board ID` | Fizzy board ID to migrate issues to |

### Linear Filters

| Option | Short | Description |
|--------|-------|-------------|
| `--team ID` | | Filter by Linear team ID |
| `--project ID` | | Filter by Linear project ID |
| `--state NAME` | | Filter by state name (e.g., "Todo", "In Progress") |
| `--label NAME` | `-l` | Filter by label (can be repeated) |
| `--issue ID` | `-i` | Migrate specific issue(s) by identifier (can be repeated) |
| `--include-archived` | | Include archived issues |
| `--limit N` | | Maximum issues to migrate (default: 100) |

### Fizzy Options

| Option | Description |
|--------|-------------|
| `--account NAME` | Fizzy account name (if not default) |
| `--column NAME` | Target Fizzy column name |

### Behavior Options

| Option | Short | Description |
|--------|-------|-------------|
| `--dry-run` | | Preview changes without executing |
| `--verbose` | `-v` | Show detailed output |
| `--quiet` | `-q` | Suppress non-error output |
| `--help` | `-h` | Show help message |
| `--version` | | Show version |

## Examples

### Migrate all open issues from a team

```bash
linear2fizzy --board 12345 --team abc123
```

### Preview migration (dry run)

```bash
linear2fizzy --board 12345 --team abc123 --dry-run --verbose
```

### Migrate specific issues

```bash
linear2fizzy --board 12345 -i PROJ-123 -i PROJ-456 -i PROJ-789
```

### Migrate issues with specific labels

```bash
linear2fizzy --board 12345 --team abc123 -l bug -l urgent
```

### Migrate to a specific column

```bash
linear2fizzy --board 12345 --team abc123 --column "Backlog"
```

### Migrate only "Todo" state issues

```bash
linear2fizzy --board 12345 --team abc123 --state "Todo"
```

### Migrate issues from a specific project

```bash
linear2fizzy --board 12345 --team abc123 --project proj_456
```

## Finding IDs

### Linear Team ID

In Linear, press `Cmd/Ctrl + K` and type "Copy model UUID" while viewing your team.

Or use the script to list teams (requires valid API key):

```bash
curl -s -X POST \
  -H "Content-Type: application/json" \
  -H "Authorization: $LINEAR_API_KEY" \
  --data '{"query":"{ teams { nodes { id name key } } }"}' \
  https://api.linear.app/graphql | jq '.data.teams.nodes'
```

### Fizzy Board ID

```bash
fizzy board list
```

### Linear Issue Identifier

Issue identifiers look like `PROJ-123` and are visible in the Linear UI and URLs.

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 2 | Invalid arguments |
| 3 | Missing dependencies |
| 4 | Authentication failure |
| 5 | Partial failure during migration |

## What Gets Migrated

For each Linear issue, the following is transferred to Fizzy:

- **Title**: Prefixed with Linear identifier (e.g., `[PROJ-123] Issue title`)
- **Description**: Full markdown description converted to HTML
- **Labels**: Converted to Fizzy tags
- **Comments**: All comments with author and timestamp attribution
- **Metadata footer**: Original state, assignee, and link back to Linear issue

## License

MIT

## See Also

- [Linear API Documentation](https://linear.app/developers)
