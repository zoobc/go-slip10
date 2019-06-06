# go-slip10

An implementation of the [SLIP-0010 spec](https://github.com/satoshilabs/slips/blob/master/slip-0010.md) for Universal private key derivation from master private key as a simple Go library. The semantics of derived keys are up to the user. [SLIP-0013](https://github.com/satoshilabs/slips/blob/master/slip-0013.md) is a good scheme to implement with this library.

## Example

It's very unlikely, but possible, that a given index does not produce a valid 
private key. Error checking is skipped in this example for brevity but should be handled in real code. In such a case, a ErrInvalidPrivateKey is returned.

ErrInvalidPrivateKey should be handled by trying the next index for a child key.

Any valid private key will have a valid public key so that `Key.PublicKey()`
method never returns an error.

```go
package main

import (
	"fmt"
	"log"

	slip10 "github.com/lmars/go-slip10"
)

// Example address creation for a fictitious company ComputerVoice Inc. where
// each department has their own wallet to manage
func main() {
	// Generate a seed to determine all keys from.
	// This should be persisted, backed up, and secured
	seed, err := slip10.NewSeed()
	if err != nil {
		log.Fatalln("Error generating seed:", err)
	}

	// Create master private key from seed
	computerVoiceMasterKey, _ := slip10.NewMasterKeyWithCurve(seed, slip10.CurveBitcoin)

	// Map departments to keys
	// There is a very small chance a given child index is invalid
	// If so your real program should handle this by skipping the index
	departmentKeys := map[string]*slip10.Key{}
	departmentKeys["Sales"], _ = computerVoiceMasterKey.NewChildKey(0)
	departmentKeys["Marketing"], _ = computerVoiceMasterKey.NewChildKey(1)
	departmentKeys["Engineering"], _ = computerVoiceMasterKey.NewChildKey(2)
	departmentKeys["Customer Support"], _ = computerVoiceMasterKey.NewChildKey(3)

	// Create public keys for record keeping, auditors, payroll, etc
	departmentAuditKeys := map[string]*slip10.Key{}
	departmentAuditKeys["Sales"] = departmentKeys["Sales"].PublicKey()
	departmentAuditKeys["Marketing"] = departmentKeys["Marketing"].PublicKey()
	departmentAuditKeys["Engineering"] = departmentKeys["Engineering"].PublicKey()
	departmentAuditKeys["Customer Support"] = departmentKeys["Customer Support"].PublicKey()

	// Print public keys
	for department, pubKey := range departmentAuditKeys {
		fmt.Println(department, pubKey)
	}
}
```

## Thanks

This library is a modified version of Tyler Smith's [go-bip32](https://github.com/tyler-smith/go-bip32) library,
much thanks goes to Tyler.

From Tyler Smith himself:

The developers at [Factom](https://www.factom.com/) have contributed a lot to this library and have made many great improvements to it. Please check out their project(s) and give them a thanks if you use this library.

Thanks to [bartekn](https://github.com/bartekn) from Stellar for some important bug catches.
