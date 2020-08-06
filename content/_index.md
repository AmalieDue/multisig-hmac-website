---
title: ""
date: 2020-08-04T09:38:32+02:00
draft: false
---

## About the project

Multisig HMAC is a novel, symmetric signature scheme allowing a threshold of signers to form a quorum. This scheme takes each separate signature and a bitfield indicating the keys used, which are combinable in any order, allowing for simple threshold schemes, or more advanced authentication flows.

Key management can happen in either of two modes, either by storing every of the component keys, or by storing a single master seed and using that to derive keys ad hoc.

Fully open-source implementations in [Javascript](#javascript), [Python](#python), and [Java](#java) and available on Github, npm, PyPI, and MavenCentral:

* [https://github.com/emilbayes/multisig-hmac](https://github.com/emilbayes/multisig-hmac) [(npm)](https://www.npmjs.com/package/multisig-hmac)
* [https://github.com/AmalieDue/multisig-hmac](https://github.com/AmalieDue/py-multisig-hmac) [(PyPI)](https://pypi.org/project/multisig-hmac/)
* [https://github.com/AmalieDue/multisig-hmac](https://github.com/AmalieDue/java-multisig-hmac) [(MavenCentral)](https://search.maven.org/artifact/dk.hyperdivision/multisig_hmac/2.0.0/jar)

---

## Javascript

Example from the Javascript implementation, using stored keys:

```javascript
const MultisigHMAC = require('multisig-hmac')

const multisigHmac = new MultisigHMAC()

// generate keys, which need to be stored securely
// and need to be shared securely with each party
const k1 = multisigHmac.keygen(1)
const k2 = multisigHmac.keygen(2)
const k3 = multisigHmac.keygen(3)

// Sign by each client with 2-of-3
const data = Buffer.from('Hello world')

// Notice no mention of nonce here. The data can follow whatever format you
// desire, but should include a nonce
const s1 = multisigHmac.sign(k1, data)
const s3 = multisigHmac.sign(k3, data)

const signature = multisigHmac.combine([s1, s3])

// Verify on the server
const threshold = 2
const keys = [k1, k2, k3]
const verified = multisigHmac.verify(keys, signature, data, threshold)
console.log(verified)
```

Example from the Javascript implementation, using a derived master key:

```javascript
const MultisigHMAC = require('multisig-hmac')

const multisigHmac = new MultisigHMAC()

// Generate a master seed, which needs to be stored securely
// This seed must NOT be shared with any other party
const seed = multisigHmac.seedgen()

const k1 = multisigHmac.deriveKey(seed, 1)
const k2 = multisigHmac.deriveKey(seed, 2)
const k3 = multisigHmac.deriveKey(seed, 3)

// Sign by each client with 2-of-3
const data = Buffer.from('Hello world')

// Notice no mention of nonce here. The data can follow whatever format you
// desire, but should include a nonce
const s1 = multisigHmac.sign(k1, data)
const s3 = multisigHmac.sign(k3, data)

const signature = multisigHmac.combine([s1, s3])

// Verify on the server, but now keys are dynamically derived
const threshold = 2
const verified = multisigHmac.verifyDerived(seed, signature, data, threshold)
console.log(verified)
```

---

## Python

Example from the Python implementation, using stored keys:
```python
import multisig_hmac
from multisig_hmac.multisig_hmac import MultisigHMAC
import base64

m = MultisigHMAC()

# generate keys which need to be stored securely and need to be shared securely with each party
k0 = m.keygen(0)
k1 = m.keygen(1)
k2 = m.keygen(2)

# sign by each client
data = b'hello world'

s0 = m.sign(k0, data)
s2 = m.sign(k2, data)

# combine the used signatures
out = m.combine([s0, s2])

sent = (out[0], base64.urlsafe_b64encode(out[1]))

# --- network ---

received = (sent[0], base64.urlsafe_b64decode(sent[1]))

# verify on the server
threshold = 2
keys = [k0, k1, k2]
signature = received

print(m.verify(keys, signature, data, threshold))
```

Example from the Python implementation, using a derived master key:

```python
import multisig_hmac
from multisig_hmac.multisig_hmac import MultisigHMAC
import base64

m = MultisigHMAC()

# generate a master seed which needs to be stored securely
# this seed must NOT be shared with any other party
seed = m.seedgen()

k0 = m.deriveKey(seed, 0)
k1 = m.deriveKey(seed, 1)
k2 = m.deriveKey(seed, 2)

# sign by each client
data = b'hello world'

s0 = m.sign(k0, data)
s2 = m.sign(k2, data)

# combine the used signatures
out = m.combine([s0, s2])

sent = (out[0], base64.urlsafe_b64encode(out[1]))

# --- network ---

received = (sent[0], base64.urlsafe_b64decode(sent[1]))

# verify on the server, but now keys are dynamically derived
threshold = 2
signature = received

print(m.verifyDerived(seed, signature, data, threshold))
```

---

## Java

Example from the Java implementation, using stored keys:
```java
package dk.hyperdivision.multisig_hmac;

import java.util.*;

public class StoredKeys {

    public static void main(String[] args) throws InvalidKeyException, NoSuchAlgorithmException {
    
        MultisigHMAC m = new MultisigHMAC(MultisigHMAC.Algorithm.HmacSHA256);

        Key k0 = m.generate(0);
        Key k1 = m.generate(1);
        Key k2 = m.generate(2);

        byte[] message = "hello world".getBytes();

        Signature s0 = m.sign(k0, message);
        Signature s2 = m.sign(k2, message);

        List<Signature> signatures = new ArrayList<>();
        signatures.add(s0);
        signatures.add(s2);
    
        Signature combined = m.combine(signatures);

        List<Key> keys = new ArrayList<>();
        keys.add(k0);
        keys.add(k1);
        keys.add(k2);
        int threshold = 2;

        System.out.println(m.verify(keys, combined, message, threshold));
    }
}
```

Example from the Java implementation, using a derived master key:

```java
package dk.hyperdivision.multisig_hmac;

import java.util.*;

public class DerivedKeys {

    public static void main(String[] args) throws NoSuchAlgorithmException, InvalidKeyException {
    
        DerivedMultisigHMAC m = new DerivedMultisigHMAC(DerivedMultisigHMAC.Algorithm.HmacSHA256);
        
        byte[] masterKey = m.generateMasterKey();

        Key k0 = m.generate(0, masterKey);
        Key k1 = m.generate(1, masterKey);
        Key k2 = m.generate(2, masterKey);

        byte[] message = "hello world".getBytes();

        Signature s0 = m.sign(k0, message);
        Signature s2 = m.sign(k2, message);

        List<Sign> signatures = new ArrayList<>();
        signatures.add(s0);
        signatures.add(s2);
        int threshold = 2;

        Signature combined = m.combine(signatures);

        System.out.print(m.verify(masterKey, combined, message, threshold));
    }
}
```

---

## Security Analysis

The cryptographic strength of the HMAC depends on the size of the secret key that is used. In this project, the maximum key sizes are used. Since the implementations support SHA256, SHA512, and SHA384 for HMAC, then key sizes of 64, 128, and 128 bytes respectively are used (i.e. 512, 1024, and 1024 bits respectively). The HMAC values have sizes 32, 64, and 48 bytes respectively (256, 512, and 384 bits respectively). In cryptography, a Pseudo Random Function family, PRF, is a collection of functions which emulate a random oracle in the following way: There exists no efficient algorithm that can distinguish between a function randomly chosen from the PRF family and a function whose outputs are completely random. In 2006, it was proved that HMAC is a PRF under the assumption that the compression function is a PRF. A way to prove that the XOR of two PRFs, or more specifically two HMACs in this case, is also PRF is by thinking about the contrapositive of the statement: If the XOR of two HMACs is not PRF, then HMAC is not PRF. The idea is to suppose that an adversary or distinguisher _D_ breaks the secrecy of the XOR'ed PRFs and then use _D_ to construct a distinguisher _D'_ which can break the security of at least one of the single PRF's. The conclusion of the proof is that 

* If two XOR'ed HMACs is not PRF, then HMAC is not PRF.
* It was proved in 2006 that HMAC is PRF.
* Hence: Two XOR'ed HMACs is PRF.

See more about the security of HMACs on the [Wikipedia page](https://en.wikipedia.org/wiki/HMAC#Security).

With an HMAC output size of 256 bits, there are N = 2^256 different HMAC values. When one HMAC value has been picked, there are N-1 HMAC values remaining. The probability of randomly picking two unique HMAC values is (N-1)/N, and likewise the probability of randomly picking the same HMAC value twice is therefore

```math
1 - (N-1)/N = N/N - (N-1)/N = (N-N+1)/N = 1/N. 
```

As the XOR'ing of two HMAC values is also PRF according to some proofs, the security level is still satisfied, i.e. the birthday bound still holds: It is possible to find a collision of HMAC values in 2^(n/2).

In conclusion, the probability that XOR'ing some number of signatures will not be successfull is negligible.