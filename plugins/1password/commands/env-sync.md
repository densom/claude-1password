---
description: Populate .env file from .env.1password template using 1Password CLI
argument-hint: [path-to-template]
---

You are tasked with generating a `.env` file by resolving 1Password secret references from a `.env.1password` template file.

## Prerequisites Check:

1. **Verify 1Password CLI is installed**: Run `op --version` to check if the 1Password CLI is available.
   - If not installed, inform the user to install it from https://developer.1password.com/docs/cli/get-started/

2. **Verify 1Password CLI is authenticated**: Run `op account list` to check if the user is signed in.
   - If not authenticated, inform the user to run `op signin` first

## Instructions:

1. **Locate the template file**: If the user provided a path argument, use that path. Otherwise, look for `.env.1password` in the current working directory.
   - If `.env.1password` doesn't exist, inform the user and suggest running `/claude-1password:env-init` first

2. **Check if .env already exists**:
   - If `.env` already exists, ask the user if they want to overwrite it
   - Consider backing up the existing `.env` as `.env.backup` before overwriting

3. **Use op inject command**: Use the 1Password CLI `op inject` command to resolve all secret references in the template file and generate the `.env` file.

   The basic command syntax is:
   ```bash
   op inject -i .env.1password -o .env
   ```
  
   The `op inject` command will:
   - Automatically find all `op://vault/item/field` references in the template
   - Resolve them by fetching the secrets from 1Password
   - Replace the references with the actual secret values
   - Preserve all comments, empty lines, and non-secret values exactly as they appear
   - Handle multiple references on a single line
   - Provide clear error messages if any references are invalid or inaccessible

If the user opted to overwrite the existing file, make sure to use the `--force` flag in the op command.

4. **Output summary**: Show the user:
   - Success message: "Successfully generated .env from .env.1password"
   - The path to the generated `.env` file
   - Number of secret references that were resolved (count `op://` occurrences in the template)
   - A reminder to add `.env` to `.gitignore` if not already present
   - Security reminder: "Remember to delete this file when no longer needed"

## Example:

Template (`.env.1password`):
```
# Database configuration
DATABASE_URL=op://Private/myapp/database_url
API_KEY=op://Private/myapp/api_key

# Non-secret values can be stored directly
NODE_ENV=development
```

Command executed:
```bash
op inject -i .env.1password -o .env
```

Generated (`.env`):
```
# Database configuration
DATABASE_URL=postgresql://user:pass@localhost:5432/db
API_KEY=sk_live_abc123xyz789

# Non-secret values can be stored directly
NODE_ENV=development
```

## Additional Options:

- **Force overwrite**: If you want to skip the overwrite confirmation, you can add the `--force` flag: `op inject -i .env.1password -o .env --force`
- **In-memory only**: To output to stdout without writing a file (useful for verification): `op inject -i .env.1password`
- **Different paths**: You can specify any template and output paths: `op inject -i path/to/template -o path/to/.env`

## Error Handling:

- If `.env.1password` doesn't exist, inform the user and suggest running `/claude-1password:env-init` first
- If `.env` already exists, ask if they want to overwrite it
- If `op inject` fails, show the specific error message from the CLI and suggest solutions:
  - **"not signed in"**: User needs to run `op signin`
  - **"item not found"** or **"vault not found"**: Check that the `op://` references in `.env.1password` match actual items in 1Password
  - **"field not found"**: Verify the field name exists in the 1Password item
  - For any errors, show which reference(s) caused the problem to help the user debug
- If the command succeeds but the output seems incorrect, suggest the user manually verify their 1Password references
