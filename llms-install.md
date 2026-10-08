# Installing the Formstep MCP server

Formstep is a hosted MCP server at `https://api.formstep.io/api/mcp`. There is nothing to clone, build or run locally.

1. Ask the user for a Formstep API token. They create one in their Formstep workspace: open **OAuth and API Keys** in the sidebar and click **Create token**. The token starts with `fs_`, reaches one workspace and expires 30 days after it is created. Docs: https://docs.formstep.io/developers/api-tokens/
2. Add this entry to `cline_mcp_settings.json`, with the user's token in place of `fs_YOUR_TOKEN`. Keep any servers that are already there.

   ```json
   {
     "mcpServers": {
       "formstep": {
         "type": "streamableHttp",
         "url": "https://api.formstep.io/api/mcp",
         "headers": {
           "Authorization": "Bearer fs_YOUR_TOKEN"
         }
       }
     }
   }
   ```

3. Check the connection by calling `workspace_list`. It returns the workspace the token belongs to.

A 401 response means the token is wrong or has expired. Ask the user to create a new one.

What to try next and the full tool list: [README.md](README.md).
