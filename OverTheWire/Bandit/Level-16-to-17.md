# OverTheWire Bandit: Level 16 → Level 17

**Focus:** Port scanning, identifying TLS services, and retrieving an SSH private key  
**Tools:** `ssh`, `nmap`, `openssl s_client`

## Objective

A service listening on one port between `31000` and `32000` returns the next level's credentials when given the current password. The first task is to identify the listening ports, then distinguish the TLS-enabled services from the plaintext services. Only one service returns the next credentials; the others are echo services.


> **Verification note:** The supplied terminal transcript confirms the port scan and TLS handshake, but it ends before the private key is returned. The `KEYUPDATE` issue is diagnosed below; run the corrected `-nocommands` command to verify the final credential response.

## Solution

### 1. Enumerate the port range

Run a version-detection scan against the specified local TCP port range:

```bash
nmap -sV -p 31000-32000 127.0.0.1
```

- `-p 31000-32000` limits the scan to the challenge's port range.
- `-sV` asks Nmap to probe open ports and identify the services where possible.
- `127.0.0.1` is the loopback address; `localhost` can be used instead.

The scan in this session found the following open ports:

| Port | Nmap result | Interpretation |
|---:|---|---|
| `31046/tcp` | `open echo` | Plain TCP echo service |
| `31518/tcp` | `open ssl/echo` | TLS-protected echo service |
| `31691/tcp` | `open echo` | Plain TCP echo service |
| `31790/tcp` | `open ssl/unknown` | TLS service whose application Nmap did not identify |
| `31960/tcp` | `open echo` | Plain TCP echo service |

The recorded follow-up scan used `-sV -AA`. The cleaner command above keeps the relevant version detection and avoids the additional aggressive-scan features. Nmap's `-A` option enables more than version detection, including OS detection, default script scanning, and traceroute; those extras are not needed for this challenge.

### 2. Interpret the service results

Two of the five open ports were identified as TLS services:

- **31518:** `ssl/echo`. This is an echo service wrapped in TLS. It returns the data sent to it, so receiving your own input back does not mean you have retrieved the next credential.
- **31790:** `ssl/unknown`. This does not mean that TLS failed. It means Nmap could not match the application behind the TLS layer to a known service signature.

The detailed Nmap output for port `31790` included the response:

```text
Wrong! Please enter the correct current password.
```

Nmap had sent service-detection probes rather than the Bandit password. That response showed that the port had an application-level password check, making it the service to investigate further.

### 3. Connect to the candidate service

The initial connection attempt reached the TLS service:

```bash
openssl s_client -connect localhost:31790
```

The TLS handshake completed. The certificate was self-signed, producing a verification warning, but `s_client` continued the handshake. The negotiated TLS connection was therefore established; the certificate warning was not the reason the password submission failed.

### 4. Prevent OpenSSL from interpreting the password as a command

The submitted `bandit16` password in the recorded terminal session began with a lowercase `k`. This is significant because OpenSSL `s_client`, when used interactively with its default command handling enabled, recognises a line beginning with `k` as a TLS 1.3 **KeyUpdate** command. Once it recognises that command, it ignores the rest of that input line instead of forwarding it to the application.

That explains the observed `KEYUPDATE` message: OpenSSL was processing the first character as one of its own connected commands. The password was not being submitted to the Bandit service as intended. The line showing the typed characters in the terminal was not proof that the remote service had echoed them.

Disable the interactive command letters with `-nocommands`:

```bash
openssl s_client -connect localhost:31790 -nocommands
```

After the TLS handshake, enter the current `bandit16` password and press Enter. The service should accept it and return the SSH private key for `bandit17`.

A shorter alternative is:

```bash
openssl s_client -quiet -connect localhost:31790
```

OpenSSL documents that `-quiet` also enables `-nocommands` and `-ign_eof`. Use `-nocommands` when you want to keep the detailed handshake output visible while disabling the special input letters.

### 5. Save the returned private key and use it for SSH

Copy the complete private-key block, including its `BEGIN` and `END` lines, into a file on the machine from which you will initiate SSH. For example:

```bash
nano bandit17.key
```

Set restrictive permissions. SSH may reject a private key file that is accessible to other users:

```bash
chmod 600 bandit17.key
```

Use the key to log in as `bandit17`:

```bash
ssh -i bandit17.key -p 2220 bandit17@bandit.labs.overthewire.org
```

The private key is the credential supplied by this level. Do not publish it or commit it to GitHub.

## What the errors and messages meant

| Message | Meaning in this session |
|---|---|
| `ssl/echo` | Nmap identified a TLS-wrapped echo service; it is not the credential service. |
| `ssl/unknown` | TLS was detected, but Nmap did not identify the application behind it. |
| `self-signed certificate` | A certificate-verification warning. `s_client` continues by default unless configured to abort on verification errors. |
| `read R BLOCK` | The interactive TLS client is waiting for more input; it is not, on its own, an error. |
| `KEYUPDATE` | OpenSSL recognised the leading `k` as its TLS 1.3 connected command rather than sending the full line as application data. |

## Key takeaways

- Scan only the range specified by the challenge instead of probing every port.
- Use `-sV` when service identification is useful; an `unknown` application label does not rule out a valid TLS service.
- Confirm a candidate by examining how it responds to application data, not only by relying on Nmap's service label.
- OpenSSL's interactive `s_client` mode reserves specific letters at the start of a line. Use `-nocommands` when the data being sent could begin with one of those letters.
- Treat retrieved SSH private keys as credentials: store them with restrictive permissions and never publish them.

## References

- [OverTheWire Bandit: Level 16 → 17](https://overthewire.org/wargames/bandit/bandit17.html)
- [OpenSSL `s_client` documentation — Connected Commands](https://docs.openssl.org/3.5/man1/openssl-s_client/)
- [Nmap documentation — Service and Version Detection](https://nmap.org/book/man-version-detection)
- [Nmap documentation — `-A` aggressive scan options](https://nmap.org/book/man-misc-options.html)
