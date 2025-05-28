
# POSTOFFICE - CTF WRITEUP

![post1](https://raw.githubusercontent.com/Juno0w0/echoCTF_Writeups/refs/heads/main/Writeups/POSTOFFICE/post1.png)

## Introduction

The **POSTOFFICE** machine from **EchoCTF** presents a challenge focused on manipulating the HTTP **POST** method. The hint tells us that although the **GET** method is commonly used to retrieve resources, in this challenge we must explore the opposite method to discover the hidden flag.

---

## Reconnaissance and Exploration

When accessing the address:

```cpp
http://10.0.41.3:1337/
```

A simple webpage is shown with the message:

> _"Everybody knows how to GET things... But can you guess the other?"_

This gives us a clear hint: the challenge is solved by using the HTTP **POST** method to send data and receive the flag.  
This can be done in two ways, which I will explain next.

---

## Execution of the Solution

A POST request was tested using the **curl** tool in the terminal:

```bash
curl -X POST http://10.0.41.3:1337/ -d "data=test"
```

- `-X POST` indicates that the HTTP POST method will be used.  
- `-d "data=test"` sends a body with the data `data=test`.

The server responded with the flag inside an HTML page:

```html
<p>Your packet has been posted. Here is your flag ETSCTF_Its_just_a_writeup_bro</p>
```

---

## Easter Egg

If we try to view the response headers, we can find the flag:

```http
curl -I http://10.0.41.3:1337/
HTTP/1.1 200 OK
Server: nginx/1.18.0
Date: Wed, 28 May 2025 20:26:28 GMT
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
X-Powered-By: PHP/7.4.23
1: ..____________
2: .< what what? >
3: . ------------
4: .........\   ^__^
5: ..........\  (oO)\_______
6: .............(__)\       )\/7: .................||----w |
8: .................||     ||
ETSCTF: ETSCTF_4928fe50c7d8d80eba74e908dd1594de
```

---

## Vulnerability Identification

- The server exposes sensitive functionality only via POST requests without authentication or validation.  
- Access to the endpoint is neither limited nor controlled for different HTTP methods.  
- It is assumed the client will send valid data with no protection against abuse or automation.

---

## Recommended Mitigation Measures

- Implement authentication and authorization for endpoints that process sensitive data.  
- Validate and sanitize inputs received via POST.  
- Limit allowed HTTP methods per endpoint according to function.  
- Monitor access and anomalous usage of endpoints with POST methods.

---
