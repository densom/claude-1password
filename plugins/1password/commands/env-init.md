---
description: Initialize a .env.1password template from an existing .env file
argument-hint: [path-to-env-file]
---

You are tasked with creating a `.env.1password` template file from an existing `.env` file.

## Instructions:

1. **Read the .env file**: If the user provided a path argument, use that path. Otherwise, look for `.env` in the current working directory.

2. **Parse the file**: Extract all environment variable key-value pairs. Ignore:
   - Empty lines
   - Lines starting with `#` (comments)
   - Malformed lines without `=`

3. **Generate template**: For each variable, create a line in the format:
   ```
   VAR_NAME=op://vault-name/item-name/field-name
   ```

   You should:
   - Keep the variable name exactly as it appears
   - For the 1Password reference, suggest a sensible structure:
     - `vault-name`: Default to "Private" or ask the user
     - `item-name`: Use a descriptive name based on the project or variable context
     - `field-name`: Use the variable name in lowercase or the actual field name in 1Password

   Example transformation:
   ```
   DATABASE_URL=postgresql://localhost:5432/mydb
   API_KEY=abc123secret
   ```

   Becomes:
   ```
   DATABASE_URL=op://Private/project-secrets/database_url
   API_KEY=op://Private/project-secrets/api_key
   ```

4. **Add security header**: Prepend or update the security comment to the top of the generated `.env.1password` file:
   ```
   # This file was auto-generated from .env.1password
   # DO NOT commit this file to version control
   # Generated on: [current date/time]

   ```
   (Note the blank line after the header)

   If the `.env.1password` template file already exists, check for the security header and add it if it does not exist.

5. **Preserve comments**: Keep any comment lines from the original file to maintain documentation.

6. **Write the template**: Save the result as `.env.1password` in the same directory as the source `.env` file.

7. **Output summary**: Show the user:
   - How many variables were converted
   - The path to the new `.env.1password` file
   - A reminder to update the 1Password references to match their actual vault structure
   - Instructions: "Edit `.env.1password` to update the `op://` references to match your 1Password vault, item, and field names. Then run `/claude-1password:env-sync` to generate your `.env` file."

## Error Handling:

- If the `.env` file doesn't exist, inform the user and ask if they want to create a new empty template
- If `.env.1password` already exists, ask if they want to overwrite it
