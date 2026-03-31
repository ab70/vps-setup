
# Docker Credential Helper Setup and Troubleshooting

## 1. Prerequisites Not Met
`docker-credential-pass` requires a fully configured pass store before it can work. This is the #1 cause of setup failures.

### Setup Checklist:
```bash
# 1. Install dependencies
sudo apt-get install gnupg2 pass  # Debian/Ubuntu
# or: brew install gnupg pass      # macOS

# 2. Generate GPG key (if you don't have one)
gpg2 --full-generate-key
# Note the key ID (e.g., A154BD21 from "pub 2048R/A154BD21")

# 3. Initialize pass store
pass init <YOUR-GPG-KEY-ID>

# 4. Install docker-credential-pass binary
wget https://github.com/docker/docker-credential-helpers/releases/download/v0.8.0/docker-credential-pass-v0.8.0.linux-amd64
chmod +x docker-credential-pass-v0.8.0.linux-amd64
sudo mv docker-credential-pass-v0.8.0.linux-amd64 /usr/local/bin/docker-credential-pass

# 5. Verify pass is working
pass insert docker-credential-helpers/docker-pass-initialized-check
pass show docker-credential-helpers/docker-pass-initialized-check
```

## 2. Docker Configuration
Your `~/.docker/config.json` should contain:
```json
{
  "credsStore": "pass"
}
```
**Important**: When using `credsStore`, Docker stores credentials in the external helper rather than in the `config.json` file itself. The `auths` section will appear empty (or with empty objects) after successful login.

## 3. Multiple Registries Support
The `docker-credential-pass` helper does support multiple registries. After logging into different registries, your pass store will look like:
```bash
$ pass
Password Store
└── docker-credential-helpers
    ├── b64-encoded-registry-1
    │   └── username1
    ├── b64-encoded-registry-2
    │   └── username2
    └── b64-encoded-registry-3
        └── username3
```

And your `config.json` will show:
```json
{
  "auths": {
    "ghcr.io": {},
    "registry.gitlab.com": {},
    "index.docker.io": {}
  },
  "credsStore": "pass"
}
```

### Login workflow for multiple registries:
```bash
# GitHub Container Registry
docker login ghcr.io -u USERNAME -p TOKEN

# GitLab Container Registry  
docker login registry.gitlab.com -u USERNAME -p TOKEN

# Docker Hub
docker login -u USERNAME -p TOKEN
```

## 4. Common Error: "pass store is uninitialized"
If you get this error even after setup, the GPG agent may have forgotten the key:
```bash
# Re-initialize by showing any pass entry
pass show docker-credential-helpers/docker-pass-initialized-check
# Enter your GPG passphrase when prompted
```

## 5. Troubleshooting Checklist

| Symptom                         | Likely Cause                        | Solution                                                                 |
|----------------------------------|-------------------------------------|--------------------------------------------------------------------------|
| executable file not found in $PATH | Binary not installed or not in PATH | Install to `/usr/local/bin` and verify `which docker-credential-pass`   |
| gpg: decryption failed: No secret key | GPG key not available or agent not running | Run `gpg2 --list-secret-keys` and ensure key exists; try `pass show` to unlock |
| pass store is uninitialized      | Pass not initialized or GPG key expired | Re-run `pass init <GPG-ID>` and test with `pass insert/show`            |
| Login prompts for password every time | Credential helper not configured   | Check `~/.docker/config.json` has `"credsStore": "pass"`                |
| Error saving credentials         | Permission issues                   | Ensure `~/.password-store` and `~/.gnupg` have correct ownership        |

## 6. Important Considerations for Multiple Registries
⚠️ **Limitation**: Docker stores credentials by registry domain only, not by full path. This means if you use different tokens for different GitLab projects (e.g., `registry.gitlab.com/group/project1` vs `registry.gitlab.com/group/project2`), Docker will overwrite the previous credential.

### Workarounds:
- Use the `--config` flag to maintain separate Docker configs:
```bash
docker --config ~/.docker-project1 login registry.gitlab.com
docker --config ~/.docker-project2 login registry.gitlab.com
```

- Use a credential manager that supports namespace isolation (consider `secretservice` on Linux or `osxkeychain` on macOS as alternatives).

## 7. Verification Commands

```bash
# Check what credentials are stored
docker-credential-pass list

# Verify a specific credential
docker-credential-pass get <<< '{"ServerURL":"https://ghcr.io"}'

# Check Docker config
cat ~/.docker/config.json | jq

# List GPG keys
gpg2 --list-secret-keys --keyid-format LONG
```