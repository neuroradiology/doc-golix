**WARNING:** This is a draft document dated May 2016. It is under review and is definitely not finalized. If you'd like to stay updated, consider joining the [Hypergolix mailing list](https://www.hypergolix.com/#page-top).

# Introduction

If Web 2.0 has been defined by the rise of the social internet, Web 3.0 will surely include the connected physical devices now being dubbed the "Internet of Things". Like its predecessors, the Internet of Things (IoT) will include and build upon the past. The emergence of social applications with physical components (currently these predominantly involve video sharing) attests to the importance of continued sharability. But a generally-accepted security solution for IoT has yet to materialize; and sites like Shodan -- which includes a search engine for vulnerable video feeds -- are yet another indication that this need is dire.

However, IoT security imposes substantially different challenges than Web 2.0. Broadly speaking there are two primary possible approaches, given existing best practices:

1. Issue certificates for each connected device, allowing end-to-end security
2. Issue certificates for a central server, requiring full trust in its owner

Even conservative growth estimates for IoT would imply that the first option, if pursued through existing certificate authority, would require orders of magnitude more certificates to be issued for IoT devices than in the entire previous history of the internet. Eschewing the certificate authorities might be possible, but with potentially dynamic IP addresses on both ends of the connection, sharability and reachability would remain substantial problems. And the second approach ultimately entails trusting companies with Facebook with access to internet-connected baby monitors. Clearly, neither option is particularly attractive.

An alternative approach would be to force end-to-end encryption of all IoT information, and implement a new addressing mechanism to communicate socially between devices. This would allow centralized, auditable services to concentrate transport-level security certificates without granting them access to data. This is an enticing compromise; however, with the requirement of scalable sharability, and more immediate constraints like dynamic data streams, existing protocols like PGP cannot fill this need.

Golix is a cryptgraphic protocol designed to fill this void. It allows for both asynchronous delivery and asynchronous sharing, making scalable encryption of data at rest possible even for highly-dynamic social systems. Critically, it requires no trust in a centralized server to achieve confidentiality of transferred data.

# Threat model

Golix protects the informational autonomy of an abstract digital ```entity-agent``` comprised of three public/private keypairs. That is to say, the Golix protocol affords these ```entities``` strict and exclusive control over:

1. creation,
2. retention,
3. and sharing

of any and all information ```possessed``` by the ```entity-agent```. Note that Golix does not include any concept of information ownership or intellectual property. Within a Golix context, information ```possession``` means only that an agent-entity retains that particular piece of information. As such, once information is shared, **the sharer's authority over the information stops at the point of share;** the recipient has explicit autonomy over their digital "memory" of that information, as does the sharer.

To enforce such agency cryptographically, Golix protects:

+ information *confidentiality* against ```illegitimate access``` by any adversary,
+ information *integrity* against deliberate or incidental modification by any adversary, and
+ information *authenticity* of authorship against impersonation by any adversary.

However, Golix provides only limited protections:

+ against meta-analysis of Golix data or traffic,
+ against flooding ```entities``` with requests, effectively (D)DoSing them, and
+ against malicious information retention or removal.

And Golix protections explicitly exclude:

+ any compromise of the underlying cryptographic primitives used by the protocol,
+ any compromise of ```entity``` (client) devices, and
+ any connection between physical entities and Golix ```entities```.

The following discusses what specific capabilities are afforded to various adversaries. Each capability builds upon (and includes) the previous category.

## *Any* adversary can...

Cryptographic compromises:

+ Use a collision of the address hash algorithm to tamper with existing information
+ Use a break of the signature algorithm to impersonate an ```entity``` in the future
+ Use a break of the public key encryption algorithm to intercept future key exchanges and compromise stored past key exchanges
+ Use a break of the symmetric encryption algorithm to expose the plaintext of past and future objects
+ Use a break of the exchange algorithm to impersonate an alias of an existing ```entity``` in the future
+ Use a break of the KDF algorithm to impersonate an alias of an existing ```entity``` in the future
+ Use a break of the exchange algorithm to forge key exchanges in the future
+ Use a break of the MAC algorithm to forge key exchanges in the future
+ Use a break of the KDF algorithm to forge key exchanges in the future

Client compromises:

+ Use a compromised client device to fully impersonate the client ```entity```
+ Use a compromised client device to gain access to the client ```entity``` private keys
+ Use a compromised client device to gain access to any client-stored content symmetric keys

Meta-analyses:

+ Tie any piece of information to exactly one ```entity```: typically the ```entity``` that created the information, but occasionally the ```entity``` that received it
+ Calculate the exact, or near-exact, length of confidential information

Denials of service:

+ Flood a ```persister``` server with *traffic*
+ Flood a client device with *messages*

## An ```entity``` (*client*) adversary can...

Denials of service:

+ Flood a ```persister``` with *messages* 

Denials of availability:

+ Maliciously prevent deletion of arbitrary data
+ Maliciously delete possessed data

## A ```persister``` (*server*) adversary can...

Meta-analyses:

+ Analyze connection traffic to infer connections between a Golix ```entity``` and its viewed ```GHID```s
+ Analyze connection information to infer physical identity, IP address, probable time zone, etc

Denials of service:

+ Maliciously take itself offline
+ Flood a client device with traffic

Denials of availability:

+ Maliciously delete arbitrary data
+ Maliciously retain arbitrary data

## A *global passive* adversary can...

Meta-analyses:

+ Analyze traffic on the transport level to tie Golix ```entities``` to physical identities
+ Analyze traffic on the transport level to infer connections between multiple Golix ```entities```

## *No* adversary can...

Client compromises:

+ No adversary can directly escalate one client compromise into another client compromise.

Meta-analyses:

+ No adversary can perform social graph analysis of ```entities```.
+ No adversary can tie object ```recipient``` entities with object ```author``` entities
+ No adversary can deduce content types, content format, or any other content metadata, without access to the content itself

# Golix overview

## Identities

Creating Golix content requires an ```identity```. This ```identity``` is used primarily for:

1. data authorship (as metadata)
2. data retention "binding" (as metadata)
3. data sharing (as a sharing endpoint)

These ```identities``` require the ability to asymmetrically sign data, receive asymmetrically encrypted data, and perform symmetric key agreement. To divide the requirements of these three separate operations, and to minimize the damage of a break against the underlying cryptographic primitives, a separate key is used for each. As such, each ```identity``` comprises of exactly three public keys:

1. The signature key, used for signing and verifying data
2. The encryption key, used to encrypt and decrypt keyshares to the identity
3. The exchange key, used for key agreement in explicitly one-to-one operations

It is crucial to note that a Golix ```identity``` has no explicit link to any physical entity. Since there are no restrictions on the creation of new ```identities```, it is therefore very important to draw a distinction between the actual capabilities of a Golix ```identity```, and those that might be expected of a physical entity. In particular, note that though the digital Golix ```entity``` explicitly lacks content deniability, a physical entity retains the ability to repudiate any links to the Golix ```identity```. Similarly, though Golix assures consistent authentication of data, it makes no claims surrounding an ```identity```'s authorization to view that data, or of any verification of that ```identity```'s claimed physical counterpart. Put simply: with Golix, deniability is wholly different from repudiation; and authentication wholly different from authorization or verification.

Furthermore, as a result of content's dynamic sharability, and the distributed, highly-asynchronous nature of social graphs, it is preferable to use long-term keypairs instead of short-term ephemeral keys: the enormous added complexity of a key update mechanism simply cannot justify the correspondingly modest gains in forward secrecy.

## Addresses

An unambiguous reference to Golix objects is critical for network operation. However, as Golix is intended to be capable of cross-server federation *without breaking addressability*, traditional URL schemes are insufficient. Furthermore, URLs have no inherent relationship with the content they represent, which results in a complicated trust relationship between URL owner and content consumer. To avoid this entirely, Golix makes use of a static, deterministic address scheme. Put simply, absolutely every Golix object is addressed by its cryptographic hash digest. This address is termed the "Golix hash identifier" or ```GHID```.

The use of hashes as an address mechanism ensures that, provided the underlying hash algorithm remains cryptographically secure, all Golix content is fundamentally non-malleable; that is to say, once created, content cannot be altered by **any** party, deliberately or accidentally. It also bears explicit mentioning that the address is constructed in an **encrypt-then-hash** order, and as such protects the confidentiality of encrypted containers from compromises and attacks of the underlying hash function.

That being said, it is obviously often useful to treat multiple sequential pieces of static content as a single dynamic object. Given a thermometer, for example, though every individual temperature reading is inherently static, the room's temperature is not. To relate the static content to dynamic concepts, Golix allows ```entity-agents``` to bind a secondary address to changing content. That way, this "dynamic" or "portable" ```GHID``` may be used to refer to changing content with an unchanging address. The security properties of this reference are discussed within the Dynamic Binding object type below.

## Example exchanges

Preface: 

1. Golix "clients" are ```identities```, and "servers" are ```persistence providers```. However, these ```identities``` and ```persistence providers``` have no direct, explicit relation to individual devices: a ```persister``` may in fact be a cloud services provider, and the ```identity``` may be simultaneously used on multiple devices.
2. There are no restrictions placed on distribution of content *in encrypted form*. Any **device** may request any Golix resource, and the ```persister``` will deliver it. Sessions between **devices** and ```persisters``` are wholly ephemeral, and any session state will always be destroyed upon disconnection. Any stateful operations must be maintained by the device client, or from within Golix objects (and therefore referencing Golix ```identities```).

Content creation:

1. An ```identity``` is initially created on the client device, and then uploaded to a ```persister```'s server(s).
2. The ```identity``` creates content within an encrypted object container
3. The ```identity``` creates a static (and/or dynamic) object binding for that container *prior to uploading the content*. This binding prevents garbage collection of the actual object, analogously to a memory-managed programming language.
4. The client device uploads the bindings to the ```persister```'s server(s).
5. The client device uploads the object container to the ```persister```'s server(s).

Content removal:

1. An ```identity``` with an existing binding creates a debinding. An ```identity``` may only remove their own bindings.
2. The client device uploads the debinding to the ```persister```'s server(s). Once validated, this clears the binding, dereferencing the original object.
3. If the original object reference count has reached zero, the object is deleted by the ```persistence provider```.
4. If the reference count is greater than zero, the client device may request a list of ```identities``` preventing the object's deletion.

Sharing content:

1. An ```identity``` creates a handshake request. Unlike other objects, which include a public reference to their *creator*, the handshake request includes a public reference to its *recipient* and a **private** reference to its creator. The handshake request also references the object to share, and includes its symmetric key.
2. The sharer's client device uploads the handshake request to the ```persister```'s server(s).
3. If another connection at the ```persistence provider``` has already subscribed to the recipient ```identity```'s ```GHID```, then that device is notified of the request.
4. Any subsequent new ```persister``` connections subscribing to the recipient ```identity```'s ```GHID``` are also notified of the handshake, until it is removed.
5. The recipient ```identity``` removes the request by debinding it. The recipient may also respond to the request with an acknowledgment (or non-acknowledgment) of success, by repeating the process in reverse, referencing the handshake request's ```GHID``` instead of the original object, and omitting the key.

# Golix network primitives

Golix network primitives comprise all stateful objects on Golix ```persisters```. Any other traffic between a client device and a ```persister``` server is ephemeral.

## Identity container (```GIDC```)

<img align="right" src="assets/primitives-gidc.png">

Identity containers combine a Golix ```entity```'s three public keys, permanently tying them together with a single static address. Identity containers are not intended to be removable from ```persisters``` and are therefore not subject to replay attacks. 

Identity containers are the concatenation of:

1. Magic number  
2. Version number  
   Note: The version number is specific to each Golix primitive type.
3. Cipher suite designation  
   Note: the cipher suite designation is an integer that corresponds to a combination of algorithms defined within the Golix spec.
4. **Signature public key**
5. **Encryption public key**
6. **Exchange public key**
7. **Resultant ```GHID```**  
   Note: the hash digest includes all of the previous bytes in the file, including the hash algorithm definition.
    1. Hash algorithm
    2. Hash digest

## Object container (```GEOC```)

<img align="right" src="assets/primitives-geoc.png">

Object containers are used to store **all** application content on Golix ```persisters```, tying it to static addresses. They are analogous to the data encapsulation schemes used by hybrid cryptosystems, but are wholly separate from key encapsulation and distribution. Because their retention lifetimes at ```persisters``` are wholly dictated by bindings, they are not subject to replay attacks. 

Object containers are the concatenation of:

1. Magic number
2. Version number
3. Cipher suite designation
4. **Author's ```GHID```**  
   Note: author ```GHID```s are the resultant ```GHID``` from the author's identity container (GIDC).
5. Payload length
6. **Symmetrically encrypted payload:**  
   ```{``` ```Arbitrary bytes``` ```}```
7. **Resultant ```GHID```**
    1. Hash algorithm
    2. Hash digest
8. **Author's signature**  
   Note: the author signs only the bytes composing the hash digest in the ```GHID```. She does not sign the entire ```GHID```.

## Static object binding (```GOBS```)

<img align="right" src="assets/primitives-gobs.png">

Static bindings prevent object garbage collection, reusing the object's original ```GHID``` for addressing (*ie* the target object is referenced directly, and not by the static binding's resultant ```GHID```). Note that, because they can be removed by their creator, they **are** potentially subject to replay attacks. As such, any debinding must be retained by the ```persister``` until and unless that debinding itself is debound. See "Persister state analysis" below for more details.

Static bindings are the concatenation of:

1. Magic number
2. Version number
3. Cipher suite designation
4. **Binding author's ```GHID```**
5. **Binding target's ```GHID```**  
   Note: similarly to author references, target references refer to the resultant ```GHID``` of the object container (GEOC) being bound.
6. **Resultant ```GHID```**
    1. Hash algorithm
    2. Hash digest
7. **Binding author's signature**

## Dynamic object binding (```GOBD```)

<img align="right" src="assets/primitives-gobd.png">

**Note: the GOBD diagram is out-of-date; see description below.**

Dynamic bindings also prevent object garbage collection, while also creating a secondary proxy address for whatever object is currently referenced by the binding. Like static bindings, they are potentially subject to replay attacks, and must follow the same mitigation process. See "Persister state analysis" below for more details.

Dynamic bindings are the concatenation of:

1. Magic number
2. Version number
3. Cipher suite designation
4. **Binding author's ```GHID```**
5. **Monotonic counter for binding frame indexing**  
   Note: the frame index monotonic counter is responsible for replay protection on dynamic bindings. They are zero-indexed. Persistence servers must reject any index smaller than the index currently existing at the server.
6. Target vector length
7. **Target vector ```GHID``` list**  
   Note: a most-recent-first sorted list of target GHIDs. The first GHID in the list is the current target. The target vector applies to only this particular frame.
8. **Resultant ```GHID```**  
   Note: the first GHID in the dynamic binding is the "dynamic" or "portable" GHID. It is defined only once -- upon finalizing the first frame of the binding. This GHID may be used as a proxy address for the most current target of the binding. Like any other GHID, it is defined as the hash digest of the entire preceding file.
    1. Hash algorithm
    2. Hash digest
9. **Resultant ```GHID```**  
   Note: the second GHID in the dynamic binding is the "static" or "frame" GHID. It is re-defined every time the binding is updated based upon the current content of the binding. It is used primarily as a reference point for the historical frame list.
    1. Hash algorithm
    2. Hash digest
10. **Binding author's signature**

## Debinding (```GDXX```)

<img align="right" src="assets/primitives-gdxx.png">

Debindings remove existing bindings, reducing their reference count and eventually freeing them for garbage collection. They may also remove existing debindings, allowing previously removed objects to be rebound and reuploaded to the ```persister```. Like bindings, they are subject to replay attacks, and so their debindings must be retained by the ```persister``` until they themselves are debound. This creates a debinding chain, the length of which will increase after every upload/delete cycle. See "Persister state analysis" below for more details.

Debindings are the concatenation of:

1. Magic number
2. Version number
3. Cipher suite designation
4. **Debinding author's ```GHID```**
5. **Debinding target's ```GHID```**
6. **Resultant ```GHID```**
    1. Hash algorithm
    2. Hash digest
7. **Debinding author's signature**

## Asymmetric request/response (```GARQ```)

<img align="right" src="assets/primitives-garq.png">

Asymmetric requests share an object with a recipient, or reply to an attempted share. They are analogous to the key encapsulation component of a hybrid cryptosystem such as PGP, but wholly separated from the data encapsulation, which is implemented by object containers. Unlike most Golix primitives, their author is not publicly available; instead, they are associated with their intended recipient, and encrypted against the recipient's public encryption key, as defined by the recipient's identity container. This results in full verification being only possible by the recipient; see "Entity state analysis" below for more details. They are removed via debinding by their recipient. A request replay attack is not particularly meaningful, but would result in ```persister``` congestion, and as such their debindings are subject to the same retention process as bindings and debindings. See "Persister state analysis" below for more details.

Asymmetric requests are the concatenation of:

1. Magic number
2. Version number
3. Cipher suite designation
4. **Request recipient's ```GHID```**
5. **Asymmetrically encrypted payload:**  
   ```{``` ```Author GHID``` ```||``` ```Target GHID``` ```||``` ```<Keyshare>``` or ```<ACK>``` or ```<NAK>``` ```}```
6. **Resultant ```GHID```**
    1. Hash algorithm
    2. Hash digest
7. **Request author's ```MAC```**  
   Note: here, both the MAC algorithm and the key agreement process are defined by the cipher suite designation. For reference, with CS-0x1 it is an HMAC performed with the key from a Diffie-Hellman exchange between the Author's and Recipient's exchange keys. This process must be carefully chosen to prevent an attacker from checking the signature against all known identities to unmask the author.

# State analysis

The criteria for acceptance or non-acceptance of Golix primitives is identical for all primitives except the asymmetric request. The request verification process varies based on the role of the verifier: the recipient ```entity```, which has privileged access to the private contents of the request, is the only party that can validate the origin and authenticity of the request. In all other cases, Golix primitives can be verified using only public information.

## ```Persister``` ("server") state analysis

As ```persisters``` are non-privileged data retention services, they can only verify Golix primitives based on their public information.

### Identity container (```GIDC```)

Identity containers must be accepted by the ```persister``` if and only if the following procedure terminates successfully:

1. Verify magic number
2. Verify version
3. Verify cipher suite
4. (The following steps may be done in parallel)
    + Verify valid signature public key
    + Verify valid encryption public key
    + Verify valid exchange public key
    + Digest remaining file
        1. Verify address algorithm
        2. Verify file hash
5. Verify EOF reached

![GIDC verification diagram](assets/operations-verify-gidc.png)

### Object container (```GEOC```)

Object containers must be accepted by the ```persister``` if and only if the following procedure terminates successfully:

1. Verify magic number
2. Verify version
3. Verify cipher suite
4. (The following steps may be done in parallel)
    + Author retrieval
        1. Verify author exists at persistence provider
        2. Retrieve author's identity file
    + Digest remaining file
        1. Verify address algorithm
        2. Verify file hash
5. Verify signature
6. Verify EOF reached

![GEOC verification diagram](assets/operations-verify-geoc.png)

### Static object binding (```GOBS```)

Static bindings must be accepted by the ```persister``` if and only if the following procedure terminates successfully:

1. Verify magic number
2. Verify version
3. Verify cipher suite
4. (The following steps may be done in parallel)
    + GOBS status check: verify no debinding exists for the binding's resultant ```GHID```
    + Binder retrieval
        1. Verify binder exists at persistence provider
        2. Retrieve binder's identity file
    + Digest remaining file
        1. Verify address algorithm
        2. Verify file hash
5. Verify signature
6. Verify EOF reached

![GOBS verification diagram](assets/operations-verify-gobs.png)

### Dynamic object binding (```GOBD```)

Dynamic bindings must be accepted by the ```persister``` if and only if the following procedure terminates successfully:

1. Verify magic number
2. Verify version
3. Verify cipher suite
4. (The following steps may be done in parallel)
    + GOBD status check: verify no debinding exists for the binding's dynamic ```GHID```
    + GOBD consistency check: 
        + if dynamic address **does not exist** at persistence provider **and** its counter is zero...
            1. Verify dynamic address algorithm
            2. Verify dynamic hash
        + elif dynamic address **does not exist** at persistence provider (but its counter is nonzero)...
            1. Dynamic address cannot be validated in this case. Optionally, the server *may* reject the binding, but *should* accept it.
        + else (binding exists and counter is nonzero):
            1. Verify existing binder matches uploaded binder
            2. Verify the frame index (the counter) increased
            3. Optionally, detect resource contentions by comparing target vector ```GHID```s after aligning them by the frame index
    + Binder retrieval
        1. Verify binder exists at persistence provider
        2. Retrieve binder's identity file
    + Digest remaining file
        1. Verify frame address algorithm
        2. Verify frame hash
5. Verify signature
6. Verify EOF reached

![GOBD verification diagram](assets/operations-verify-gobd.png)

### Debinding (```GDXX```)

Debindings must be accepted by the ```persister``` if and only if the following procedure terminates successfully:

1. Verify magic number
2. Verify version
3. Verify cipher suite
4. (The following steps may be done in parallel)
    + GDXX status check: verify no debinding exists for the debinding's resultant ```GHID```
    + GDXX reference check
        1. Verify target GHID exists at persistence provider
        2. Verify debinder GHID matches binder GHID (for GOBS, GOBD, GDXX) or recipient GHID (for GARQ)
    + Binder retrieval
        1. Verify binder exists at persistence provider
        2. Retrieve binder's identity file
    + Digest remaining file
        1. Verify address algorithm
        2. Verify file hash
5. Verify signature
6. Verify EOF reached

![GDXX verification diagram](assets/operations-verify-gdxx.png)

### Asymmetric request/response (```GARQ```)

Asymmetric requests must be accepted by the ```persister``` if and only if the following procedure terminates successfully:

1. Verify magic number
2. Verify version
3. Verify cipher suite
4. (The following steps may be done in parallel)
    + Verify recipient exists at persistence provider
    + Digest remaining file
        1. Verify address algorithm
        2. Verify file hash
5. Verify EOF reached

![GARQ persister verification diagram](assets/operations-verify-garq_server.png)

## ```Entity``` ("client") state analysis

```Entities``` should independently verify all state analyses performed by the ```persister```. They should also perform additional privileged verification of the asymmetric request/response primitive.

### Asymmetric request/response (```GARQ```)

Asymmetric requests must be accepted by the ```entity``` if and only if the following procedure terminates successfully:

1. Verify magic number
2. Verify version
3. Verify cipher suite
4. Digest remaining file
    1. Verify address algorithm
    2. Verify file hash
5. Verify EOF reached
6. Decrypt payload
7. Author verification
    1. Verify author exists
    2. Retrieve author's identity file
8. Perform symmetric signature key agreement procedure (see Appendix B below)
9. Verify author symmetric signature

**Note that for the recipient, rejection should imply an immediate debinding of the GARQ,** optionally accompanied by a NAK response to the request's author, sent by a responding (second) GARQ.

![GARQ recipient verification diagram](assets/operations-verify-garq_client.png)

# Conclusion

Because all Golix content is encrypted, it can be distributed freely, without regard to access credentials -- or, in a practical and immediate sense, with complete disregard to any session information. Though this renders the security of Golix information wholly dependent upon the protocol itself, such a relationship is, realistically speaking, already a practical reality for the vast majority of contemporary internet traffic via TLS.

By embracing encryption as the sole source of confidentiality and treating all information as private by default, and then separating cryptographic content access from transactional content delivery, Golix effectively reduces the problem of access-restricted social sharing to the far simpler task of key encapsulation and distribution. In so doing, Golix allows centralized or federated servers to be used for asynchronous object persistence, without relying on them for an expectation of secrecy.

These Golix ```persisters``` use Golix ```identities``` to distinguish ephemeral connection verbs (like address subscription and object retrieval) from state-mutating verbs (object creation or sharing). By forcing all mutating requests to use Golix primitives, Golix ```persisters``` can limit all connections to either ephemeral, nullipotent requests (ex: get, subscribe), or a single state-mutating request (publish). Furthermore, as all Golix content is hash-addressed, publishing a Golix primitive to a ```persister``` is guaranteed idempotent.

In many ways, the critical insight of Golix is this *bidirectional* minimization of trust between client and server. Clients need not trust servers for confidentiality, and servers need not trust clients for authentication. Both clients and servers, operating in their capacities as ```entities``` and ```persisters``` respectively, are able to function independently. Provided the Golix primitives pass validation on the server, the ```persister``` is assured of authorized content, and provided the primitive is constructed properly on the client, the ```entity``` is assured of confidentiality.

This strict division of concerns is especially advantageous to the emerging Internet of Things, where traditional trust infrastructure does not map well to its forecasted hyper-exponential growth. Importantly, it maintains the dynamic sharability that has become a hallmark of the social web, clearing the way for easy integration of these networked physical devices with a diverse range of social applications.

# Appendix A: Glossary

## GHID

The "Golix hash identifier": a unique content-based static address for content on a Golix-compliant network or device.

## Entity-agent

An abstract digital entity represented by three linked public/private keypairs and a ```GHID```. Also informally referred to as an agent, entity, or identity. Entity-agents are the Golix equivalent of a single consistent consciousness. **The Golix protocol does not inherently relate Golix entities with physical entities.** Any such connection must be made by applications.

## Possession

Within a Golix context, information possession is wholly unrelated to information authorship or any notion of intellectual property. Here, to possess information means only that a particular entity-agent retains it.

## Illegitimate access

The access of information by any party with whom that information has not been affirmatively, explicitly shared. This party is an arbitrary adversary, who is not limited to attacks within the Golix protocol.

## Persistence providers

The servers that physically store Golix data. Also referred to as persisters, persistence, and occasionally (and highly informally) servers.

# Appendix B: Cryptographic primitives

## Ciphersuite ```0x1```

+ **Asymmetric signature:** RSASSA-PSS, MGF1+SHA512, public exponent 65537. Salt length 64 bytes.
+ **Asymmetric encryption:** RSAES-OAEP, MGF1+SHA512, public exponent 65537.
+ **Symmetric encryption:** AES-256 in CTR mode. Nonce distributed privately, with the key.
+ **Symmetric shared secrets:** ECDH over Curve25519.
+ **Key agreement from shared secret:** HKDF+SHA512. Salt with bitwise XOR of the the address components of the two parties' GHIDs.
+ **```GARQ``` MAC:** HMAC+SHA512

### Key/nonce serialization

The serialization of the AES-CTR nonce and key (which are contained within asymmetric ```GARQ``` primitives) is as follows:

| Offset | Length    | Name                       | Format                         |
| ------ | --------- | -------------------        | -----------                    |
| 0      | 2B        | Magic number               | ASCII "```SH```"               |
| 2      | 2B        | Key servialization version | Unsigned 16-bit int            |
| 4      | 1B        | Cipher suite identifier    | Unsigned 8-bit int: ```0x01``` |
| 5      | 32B       | Key                        | Bytes                          |
| 37     | 16B       | Nonce                      | Bytes                          |

Note: the current secret serialization version is 2.

### Full ```GARQ``` MAC procedure

From a resultant GHID ```rGHID``` that was already calculated during request creation, MAC calculation (which uses the recipient's and author's ```exchange keys```, is performed as follows:

```
ghidA = bytes(65)
addrA = ghidA[0:1] = 0x01
saltA = ghidA[1:65]

ghidB = bytes(65)
addrB = ghidA[0:1] = 0x01
saltB = ghidA[1:65]

salt = saltA XOR saltB
shared = ECDH(KA, KB)
key = HKDF-SHA512(shared, salt)

A → B: HMAC-SHA512(key, rGHID)
```

![MAC generation diagram](assets/operations-mac-cs1.png)