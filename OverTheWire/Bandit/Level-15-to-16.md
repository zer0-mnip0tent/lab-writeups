# OverTheWire Bandit: Level 15 → Level 16

**Focus:** Establishing an SSL/TLS connection and sending application data through it  
**Tools:** `ssh`, `openssl s_client`

## Objective

The next level's password is returned by a service listening on `localhost:30001`. The current level's password must be submitted over an SSL/TLS connection rather than a plain TCP connection.

Official challenge: <https://overthewire.org/wargames/bandit/bandit16>

## Solution

### 1. Choose a TLS client

The service is already running on the Bandit host, so the task is to connect to it as a client. OpenSSL's `s_client` subcommand is designed for this purpose.

`s_server` would be the wrong direction: it starts a TLS server and waits for clients. `s_client` initiates a connection to an existing TLS service.

### 2. Connect to the service

```bash
openssl s_client -connect localhost:30001
```

The `-connect` option takes the destination as **one argument** in `host:port` format. In this challenge, that value is `localhost:30001`.

Once the TCP connection is established, `s_client` performs the TLS handshake. It prints connection and certificate details, then accepts input from the terminal. A line such as `read R BLOCK` is not, by itself, a connection failure; at this point the client is waiting for application data.

### 3. Submit the current password

At the interactive prompt, enter the password for `bandit15` and press Enter. The service validates the submitted value and returns a success message followed by the password for `bandit16`.

Do not include the password in a public writeup, screenshot, shell transcript, or Git commit. Use a placeholder when documenting the interaction.

## Why this works

- `nc` creates a raw TCP connection. It does not perform the TLS handshake required by this service.
- `openssl s_client` opens the TCP connection and negotiates TLS before forwarding application data.
- The password is sent as ordinary application data inside the encrypted TLS connection. It does not need to be encrypted manually.

## Troubleshooting notes

### `-connect` format

The destination must be supplied as a single `host:port` argument:

```bash
openssl s_client -connect localhost:30001
```

Do not split the host and port across two `-connect` options, and do not provide only the port number as the destination.

### `s_client` versus `s_server`

Use `s_client` when connecting to a service that is already listening. `s_server` is for creating a TLS server of your own.

### `KEYUPDATE`, `RENEGOTIATING`, or other connected commands

In interactive mode, `s_client` recognises certain single letters at the beginning of an input line as local commands. In particular, `k` and `K` trigger TLS 1.3 key-update operations. If a password begins with one of these command letters, OpenSSL may interpret the line as a command instead of sending it to the service.

To disable that command handling, use `-nocommands`:

```bash
openssl s_client -connect localhost:30001 -nocommands
```

Alternatively, `-quiet` also disables the interactive commands, while suppressing most connection and certificate details.

## Key takeaways

- A port accepting TCP connections does not imply that it accepts plaintext application data.
- TLS is negotiated before the application-level password is transmitted.
- `openssl s_client -connect host:port` is a useful way to inspect and interact with TLS services.
- Read the `s_client` manual when its interactive control messages appear; some input lines can be interpreted as OpenSSL commands rather than payload.

## References

- [OverTheWire Bandit: Level 15 → 16](https://overthewire.org/wargames/bandit/bandit16)
- [OpenSSL `s_client` documentation](https://docs.openssl.org/3.5/man1/openssl-s_client/)
