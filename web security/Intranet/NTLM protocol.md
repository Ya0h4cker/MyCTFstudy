# NTLM protocol

The NTLM protocol is a computer network authentication protocol, which is mainly used in Windows NT and Windows 2000 Server (or later) work group environment.

## Authentication exchange

The NTLM is based on challenge-response principle. There are three steps in this exchange:

- **Negotiation**: The client tells the server that it wants to authenticate to it.

- **Challenge**: The server sends a challenge to the client. This is a random value that changes with each authentication request.

- **Response**: The client encrypts the previously received challenge using a hashed version of its password as the key, and returns this encrypted version to the server, along with its username and possibly its domain.

To finalize the authentication, the server encrypts the challenge using the hash of the hashed version of client key and compare it with the response.

## Some conceptions

- NTLM hash: it's the hash versions of user passwords and divided into NT hash and LM hash. LM hashes are totally obsolete.

- Net-NTLM hash: it's used to refer to the challenge response set by the client.
