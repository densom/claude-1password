---
description: Add a new environment variable to .env.1password template and optionally to 1Password
argument-hint: <VAR_NAME> [op-reference-or-value]
---

You are tasked with adding a new environment variable to the `.env.1password` template file and optionally creating or updating the corresponding secret in 1Password.

## Instructions:

1. **Parse arguments**:
   - First argument (required): The variable name (e.g., `API_KEY`)
   - Second argument (optional): Either:
     - A 1Password reference like `op://Private/myapp/api_key`
     - A value to be stored in 1Password
     - If not provided, ask the user for input

2. **Check for .env.1password**: Look for the template file in the current working directory.
   - If it doesn't exist, ask if the user wants to create it (and offer to run `/claude-1password:env-init` if a `.env` exists)

3. **Check if variable already exists**:
   - If the variable name already exists in `.env.1password`, ask if they want to update it
   - Show the current value/reference before updating

4. **Determine the 1Password reference**:

   **Option A**: If the second argument looks like a 1Password reference (`op://...`):
   - Use it as-is
   - Verify it's valid by trying `op read "reference"` (optional, warn if it fails)

   **Option B**: If the second argument is a value (not starting with `op://`):
   - Ask the user for the 1Password item details:
     - Vault name (default: "Private")
     - Item name (suggest based on project or existing pattern)
     - Field name (default: variable name in lowercase)
   - Offer to create the item/field in 1Password using the CLI:
     ```bash
     op item create --category=password --title="item-name" \
       --vault="vault-name" field-name="value"
     ```
     Or if the item exists, update it:
     ```bash
     op item edit "item-name" --vault="vault-name" field-name="value"
     ```
   - Construct the reference: `op://vault/item/field`

   **Option C**: If no second argument provided:
   - Ask the user whether they want to:
     1. Provide an existing 1Password reference
     2. Provide a value to store in 1Password
     3. Leave it as a placeholder for manual editing later

5. **Update .env.1password**:
   - Add the new variable in the format: `VAR_NAME=op://vault/item/field`
   - Optionally add a comment above it if the user wants to document it
   - Maintain alphabetical order or append to the end (ask user preference)

6. **Optionally update .env**:
   - Ask if they want to immediately sync to `.env` by running `/claude-1password:env-sync`
   - Or just update `.env.1password` for now

7. **Output summary**: Show the user:
   - The variable that was added: `VAR_NAME=op://...`
   - Whether it was also created/updated in 1Password
   - Next steps (e.g., "Run `/claude-1password:env-sync` to update your `.env` file")

## Examples:

### Example 1: Add with existing 1Password reference
```
/claude-1password:env-add NEW_API_KEY op://Private/myapp/new_api_key
```

Result in `.env.1password`:
```
NEW_API_KEY=op://Private/myapp/new_api_key
```

### Example 2: Add with a new value
```
/claude-1password:env-add STRIPE_KEY sk_test_abc123
```

You would then:
1. Ask for vault/item/field details (or use smart defaults)
2. Create/update in 1Password
3. Add to `.env.1password` with the generated reference

### Example 3: Add placeholder for manual editing
```
/claude-1password:env-add DATABASE_PASSWORD
```

You would prompt for what to do, and potentially add:
```
DATABASE_PASSWORD=op://Private/myapp/database_password
```

## Error Handling:

- If 1Password CLI is not installed or not authenticated, warn the user but still allow adding to `.env.1password` as a placeholder
- If variable name is invalid (contains spaces, special chars except `_`), suggest a corrected name
- If 1Password operation fails, show the error but still update the template file
