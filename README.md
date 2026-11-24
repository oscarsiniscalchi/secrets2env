# secret2env

Download an AWS Secrets Manager secret into a local `.env` file.

`secret2env` reads a secret that stores its value as JSON key/value pairs and writes each pair as a `KEY="value"` line to `.env.<timestamp>` in the current directory.

## Requirements

- [AWS CLI v2](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
- [jq](https://jqlang.github.io/jq/)
- An active AWS session with `secretsmanager:GetSecretValue` permission on the secret

On macOS, install the dependencies with Homebrew:

```bash
brew install awscli jq
```

## Installation

Clone this repository, then install the script into a directory on your `PATH`.

### Option 1: System-wide (all users, all shells)

`/usr/local/bin` is on the default `PATH` for zsh, bash, and fish on macOS.

```bash
sudo mkdir -p /usr/local/bin
sudo cp secret2env /usr/local/bin/secret2env
sudo chmod 755 /usr/local/bin/secret2env
```

To pick up future changes from this repository automatically, create a symlink instead of copying:

```bash
chmod 755 secret2env
sudo ln -sf "$PWD/secret2env" /usr/local/bin/secret2env
```

### Option 2: Current user only (no sudo)

```bash
mkdir -p ~/.local/bin
cp secret2env ~/.local/bin/secret2env
chmod 755 ~/.local/bin/secret2env
```

Add `~/.local/bin` to your `PATH` for each shell you use:

```bash
# zsh
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc

# bash (macOS login shells read .bash_profile)
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bash_profile

# fish
fish -c 'fish_add_path ~/.local/bin'
```

### Verify the installation

Open a new terminal and run:

```bash
which secret2env
secret2env --help
```

## Usage

```text
secret2env [secret-name] [region]
```

| Argument      | Required | Default     | Description                                                   |
| ------------- | -------- | ----------- | ------------------------------------------------------------- |
| `secret-name` | No       | (prompted)  | Name or ARN of the secret. You are prompted if it is omitted. |
| `region`      | No       | `us-east-1` | AWS region the secret lives in.                               |

### Examples

Prompt for the secret name:

```bash
cd ~/projects/my-app
secret2env
# AWS Secrets Manager secret name: my-app/dev
# Saved 12 variables to /Users/you/projects/my-app/.env.20250114-153012
```

Pass the secret name directly:

```bash
secret2env my-app/dev
```

Use a different region:

```bash
secret2env my-app/dev us-west-2
```

Use a specific AWS profile:

```bash
AWS_PROFILE=my-profile secret2env my-app/dev
```

## Output

Given a secret with this value:

```json
{
  "DATABASE_HOST": "db.example.com",
  "DATABASE_PORT": 5432,
  "FEATURE_FLAG": true
}
```

`secret2env` creates a file named `.env.YYYYMMDD-HHMMSS`:

```bash
DATABASE_HOST="db.example.com"
DATABASE_PORT="5432"
FEATURE_FLAG="true"
```

To use it as your active environment file, rename or copy it:

```bash
cp .env.20250114-153012 .env
```

## Behavior

- The file is written to the current working directory.
- The file is created with `600` permissions, so only your user can read it.
- If the AWS call fails (expired session, wrong name, missing permission), no file is created.
- If the secret is not a JSON object (for example, a plain-text secret), the script exits with an error.

## Limitations

- Values are wrapped in double quotes without escaping. A value that contains a `"` or a newline produces a line that most dotenv parsers cannot read.
- Nested JSON values are written as JSON strings on a single line.
- Binary secrets (`SecretBinary`) are not supported.

## Security

The generated files contain secrets. Do not commit them.

Add this to the `.gitignore` of any project where you run `secret2env`:

```gitignore
.env
.env.*
```

Delete old `.env.<timestamp>` files when you no longer need them.

## Uninstall

```bash
# System-wide install
sudo rm /usr/local/bin/secret2env

# Current-user install
rm ~/.local/bin/secret2env
```
