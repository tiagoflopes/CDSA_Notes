HTTPs traffic is encrypted. As such, knowing the indicators and signs of malicious HTTPs traffic is crucial to our traffic analysis efforts.

## HTTPs Breakdown

It is a stateful protocol which incorporates encryption to provide security for web servers and client. It does so with the following:
- Transport Layer Security (TLS)
- Secure Sockets Layer (SSL)

Generally speaking, when a client establishes a HTTPs connection with a server, it conducts the following:
1. **Handshake** - The client and server agree upon which encryption algorithms to use, and exchange their certificates.
2. **Encryption** - Encrypt further data communicated between them.
3. **Further Data Exchange** - Continue to exchange data (web pages, images or other web resources).
4. **Decryption** - They decrypt this data with the private and public keys.

As such, one of the more common HTTPs based attacks are SSL renegotiation, in which the attacker negotiates the session to the lowest possible encryption standard.
There's also the [The Heartbleed](https://heartbleed.com/) attack.

## TLS and SSL Handshakes

![[tls-ssl-handshake.png]]
First, there's the classic 3-way TCP handshake. Then, we might observe the following occur during our traffic analysis efforts:
1. **Client Hello** - This is the first message and contains information like what TLS/SSL versions are supported by the client, a list of cipher suites, and random data (nonces) to be used in the following steps.
2. **Server Hello*** - In response to the client, the servers sends this message, that includes the server's chosen TLS/SSL version, its selected cipher suite, and an additional nonce.
3. **Certificate Exchange** - The server sends its digital certificate to the client, proving its identity. This includes the server's public key, which the client will use to conduct the key exchange process.
4. **Key exchange** - The client generates the premaster secret, encrypts it using the server's public key and sends it on to the server.
5. **Session Key Derivation** - Then, with the nonces exchanged in the first 2 steps, along with the premaster secret, both compute the session keys. These are used for symmetric encryption and decryption.
6. **Finished Messages** - These are sent to verify that the handshake is completed and successful, and also that both parties have derived the same session keys. This message contains the hash of all previous handshake messages and is encrypted using the session keys.
7. **Secure Data Exchange** - It's time to exchange data over the encrypted channel.

Or from a general algorithmic perspective.
![[tls_ssl_algorithmic_view.png]]

## Dividing into SSL Renegotiation Attacks

When we are looking for SSL renegotiation attacks, we can look for the following:
- **Multiple Client Hellos** - In order to get the lowest possible cipher suite.
- **Out of Order Handshake Messages** - Like receiving a client hello after completion of the handshake (although this could also happen due to packet loss).

This attack is performed to:
- **Denial of Service** - Since these consume a ton of resources on the server side.
- **SSL/TLS Weakness Exploitation** - To exploit vulnerabilities with our current implementation of cipher suites.
- **Cryptanalysis** - To analyze our SSL/TLS patterns for other systems.
