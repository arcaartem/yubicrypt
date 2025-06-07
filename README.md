# YubiCrypt

A command-line tool that uses your YubiKey's FIDO2 SSH keys for secure text encryption and decryption.

## Overview

YubiCrypt leverages existing FIDO2 SSH keys (like those generated for GitHub authentication) to encrypt and decrypt text data. This provides a convenient way to secure sensitive information using hardware-backed cryptography without needing separate encryption keys.

## Features

- 🔐 **Hardware-backed encryption** using YubiKey FIDO2 SSH keys
- 🚀 **No additional key management** - uses your existing SSH keys
- 💻 **Command-line interface** for easy integration into scripts and workflows
- 🔒 **AES-256-CBC encryption** with SSH signature-derived keys
- 📝 **Flexible input/output** - supports stdin/stdout and command-line arguments
- ✅ **Comprehensive validation** of SSH keys and YubiKey presence

## Requirements

- **YubiKey** with FIDO2 support
- **FIDO2 SSH key** (ecdsa-sk or ed25519-sk)
- **Dependencies:**
  - `openssl`
  - `ssh-keygen`
  - `ykman` (YubiKey Manager CLI)

### Installing Dependencies

**macOS (Homebrew):**
```bash
brew install openssl ykman
```

**Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install openssl yubikey-manager
```

**Fedora/RHEL:**
```bash
sudo dnf install openssl yubikey-manager
```

## Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/yourusername/yubicrypt.git
   cd yubicrypt
   ```

2. **Make the script executable:**
   ```bash
   chmod +x yubicrypt
   ```

3. **Optionally, add to your PATH:**
   ```bash
   sudo ln -s $(pwd)/yubicrypt /usr/local/bin/yubicrypt
   ```

## Setup

### Creating a FIDO2 SSH Key

If you don't already have a FIDO2 SSH key, create one:

```bash
# ECDSA variant (recommended)
ssh-keygen -t ecdsa-sk -f ~/.ssh/id_ecdsa_sk -O resident

# Or Ed25519 variant
ssh-keygen -t ed25519-sk -f ~/.ssh/id_ed25519_sk -O resident
```

**Note:** The `-O resident` flag stores the key on your YubiKey, making it portable across devices.

## Usage

### Basic Commands

```bash
# Encrypt text (output to stdout)
./yubicrypt -k ~/.ssh/id_ecdsa_sk encrypt "My secret password"

# Encrypt from stdin
echo "My secret data" | ./yubicrypt -k ~/.ssh/id_ecdsa_sk encrypt

# Decrypt from command line
./yubicrypt -k ~/.ssh/id_ecdsa_sk decrypt "challenge:iv:encrypted_data"

# Decrypt from stdin
echo "challenge:iv:encrypted_data" | ./yubicrypt -k ~/.ssh/id_ecdsa_sk decrypt
```

### Using Environment Variables

Set the SSH key file once:
```bash
export SSH_KEY_FILE=~/.ssh/id_ecdsa_sk

# Now you can omit the -k flag
./yubicrypt encrypt "My secret"
echo "encrypted_data" | ./yubicrypt decrypt
```

### Practical Examples

**Encrypt a password for storage:**
```bash
./yubicrypt -k ~/.ssh/id_ecdsa_sk encrypt "my_database_password" > password.enc
```

**Decrypt when needed:**
```bash
DB_PASSWORD=$(cat password.enc | ./yubicrypt -k ~/.ssh/id_ecdsa_sk decrypt)
```

**Encrypt files:**
```bash
cat sensitive_file.txt | ./yubicrypt -k ~/.ssh/id_ecdsa_sk encrypt > sensitive_file.enc
```

**Integration with scripts:**
```bash
#!/bin/bash
export SSH_KEY_FILE=~/.ssh/id_ecdsa_sk

# Store encrypted API key
API_KEY=$(echo "sk-1234567890abcdef" | ./yubicrypt encrypt)

# Later, decrypt for use
DECRYPTED_KEY=$(echo "$API_KEY" | ./yubicrypt decrypt)
curl -H "Authorization: Bearer $DECRYPTED_KEY" https://api.example.com/data
```

## How It Works

1. **Challenge Generation:** A random challenge is created for each encryption operation
2. **SSH Signing:** Your YubiKey signs the challenge using the FIDO2 SSH key (requires touch)
3. **Key Derivation:** The signature, challenge, and key fingerprint are combined and hashed to derive an AES-256 key
4. **Encryption/Decryption:** Standard AES-256-CBC encryption is performed using the derived key

This approach ensures that:
- Each encryption uses a unique key (derived from a unique challenge)
- The YubiKey must be present and touched for both encryption and decryption
- No long-term encryption keys are stored on disk

## Security Considerations

- **Physical Security:** Your YubiKey must be physically present for all operations
- **Touch Requirement:** Each operation requires a physical touch of the YubiKey
- **No Key Storage:** Encryption keys are derived on-demand and never stored
- **Forward Secrecy:** Each encryption operation uses a unique derived key
- **SSH Key Security:** Protect your SSH private key file with appropriate permissions (`chmod 600`)

## Troubleshooting

### Common Issues

**"YubiKey not detected"**
- Ensure your YubiKey is inserted
- Try running `ykman info` to verify detection

**"SSH key is not a FIDO2 security key"**
- Make sure you're using an SSH key created with `-sk` suffix
- Verify with: `ssh-keygen -l -f ~/.ssh/your_key`

**"Failed to sign challenge"**
- Ensure you touch the YubiKey when it blinks
- Check that the SSH key file has correct permissions
- Verify the YubiKey has FIDO2 enabled: `ykman fido info`

**"Permission denied" errors**
- Set correct permissions: `chmod 600 ~/.ssh/id_ecdsa_sk`
- Ensure you own the SSH key file

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request. For major changes, please open an issue first to discuss what you would like to change.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- Built for use with YubiKey FIDO2 security keys
- Inspired by the need for simple, hardware-backed encryption
- Uses standard OpenSSL and SSH tools for maximum compatibility 