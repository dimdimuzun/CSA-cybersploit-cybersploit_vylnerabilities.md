
# List of Identified Vulnerabilities in CTF MoneyBox

## 1. Anonymous FTP Access (Port 21)
- Access is allowed without a password.
- Enables viewing and downloading files.

## 2. Storage of Sensitive Data in FTP
- The open directory contains an image with steganographically hidden data.

## 3. Hidden Directories and Hints in HTML Source Code
- Hidden directories `blogs` and `S3ct3r-T3xt` are accessible.
- The key `3xtr4ctd4t4` is present in the page’s source code.

## 4. Steganographically Hidden Confidential Data
- The image contains a hidden file `data.txt` with information about the user `renu`.
- The extracted data includes a message indicating a weak password.

## 5. Weak User Password (Bruteforced Using rockyou.txt)
- The password `987654321` is easily cracked using Hydra.

## 6. Unsafe SSH Configuration
- User `renu` can log in via SSH without additional restrictions.
- User `renu`'s public key is stored in another user's `.ssh/authorized_keys` — `lily`.

## 7. Unsafe sudo Privileges
- User `lily` can execute `sudo perl` without entering a password.
- This allows privilege escalation using:
  ```sh
  sudo perl -e 'exec "/bin/bash";'
  ```

## 8. Storing Flags and Sensitive Data in Accessible Home Directories
- Users can access each other’s files due to misconfigured permissions.
- Sensitive files are stored in open directories.

