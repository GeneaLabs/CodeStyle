# .env
## Do
- store sensitive settings (passwords, API keys, etc.) in the `.env` file.

## Do Not
- commit the `.env` file to version control (Git).
- use the `env()` command outside of config files, as it calls the cached config values, and not the actual values in the `.env` file.
