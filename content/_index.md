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