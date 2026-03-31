# 1. Install dependencies
```
sudo apt-get install gnupg2 pass  # Debian/Ubuntu
```
# or: brew install gnupg pass      # macOS

# 2. Generate GPG key (if you don't have one)
```
gpg2 --full-generate-key
```
# Note the key ID (e.g., A154BD21 from "pub 2048R/A154BD21")

# 3. Initialize pass store
```
pass init <YOUR-GPG-KEY-ID>
```
# 4. Install docker-credential-pass binary
```
wget https://github.com/docker/docker-credential-helpers/releases/download/v0.8.0/docker-credential-pass-v0.8.0.linux-amd64
```
```
chmod +x docker-credential-pass-v0.8.0.linux-amd64
```

```
sudo mv docker-credential-pass-v0.8.0.linux-amd64 /usr/local/bin/docker-credential-pass
```


# 5. Verify pass is working
```
pass insert docker-credential-helpers/docker-pass-initialized-check
pass show docker-credential-helpers/docker-pass-initialized-check
```