# Golang (GO) Javascript Object Signing and Encryption (JOSE) and JSON Web Token (JWT) implementation

Pure Golang (GO) library for generating, decoding and encrypting [JSON Web Tokens](http://tools.ietf.org/html/draft-jones-json-web-token-10). Zero dependency, relies only
on standard library.

Supports full suite of signing, encryption and compression algorithms defined by [JSON Web Algorithms](https://tools.ietf.org/html/draft-ietf-jose-json-web-algorithms-31) as of July 4, 2014 version.

Extensively unit tested and cross tested (100+ tests) for compatibility with [jose.4.j](https://bitbucket.org/b_c/jose4j/wiki/Home), [Nimbus-JOSE-JWT](https://bitbucket.org/nimbusds/nimbus-jose-jwt/wiki/Home), [json-jwt](https://github.com/nov/json-jwt) and
[jose-jwt](https://github.com/dvsekhvalnov/jose-jwt) libraries. 


##Status
Used in production. GA ready.

## Supported JWA algorithms

**Signing**
- HMAC signatures with HS256, HS384 and HS512.
- RSASSA-PKCS1-V1_5 signatures with RS256, RS384 and RS512.
- RSASSA-PSS signatures (probabilistic signature scheme with appendix) with PS256, PS384 and PS512.
- ECDSA signatures with ES256, ES384 and ES512.
- NONE (unprotected) plain text algorithm without integrity protection

**Encryption**
- RSAES OAEP (using SHA-1 and MGF1 with SHA-1) encryption with A128CBC-HS256, A192CBC-HS384, A256CBC-HS512, A128GCM, A192GCM, A256GCM
- RSAES OAEP 256 (using SHA-256 and MGF1 with SHA-256) encryption with A128CBC-HS256, A192CBC-HS384, A256CBC-HS512, A128GCM, A192GCM, A256GCM
- RSAES-PKCS1-V1_5 encryption with A128CBC-HS256, A192CBC-HS384, A256CBC-HS512, A128GCM, A192GCM, A256GCM
- A128KW, A192KW, A256KW encryption with A128CBC-HS256, A192CBC-HS384, A256CBC-HS512, A128GCM, A192GCM, A256GCM
- A128GCMKW, A192GCMKW, A256GCMKW encryption with A128CBC-HS256, A192CBC-HS384, A256CBC-HS512, A128GCM, A192GCM, A256GCM
- ECDH-ES with A128CBC-HS256, A192CBC-HS384, A256CBC-HS512, A128GCM, A192GCM, A256GCM
- ECDH-ES+A128KW, ECDH-ES+A192KW, ECDH-ES+A256KW with A128CBC-HS256, A192CBC-HS384, A256CBC-HS512, A128GCM, A192GCM, A256GCM
- PBES2-HS256+A128KW, PBES2-HS384+A192KW, PBES2-HS512+A256KW with A128CBC-HS256, A192CBC-HS384, A256CBC-HS512, A128GCM, A192GCM, A256GCM
- Direct symmetric key encryption with pre-shared key A128CBC-HS256, A192CBC-HS384, A256CBC-HS512, A128GCM, A192GCM and A256GCM

**Compression**
- DEFLATE compression

## Installation
### Grab package from github
`go get github.com/dvsekhvalnov/jose2go` or `go get -u github.com/dvsekhvalnov/jose2go` to update to latest version

### Import package
	import (
		"github.com/dvsekhvalnov/jose2go"
	)

## Usage
#### Creating Plaintext (unprotected) Tokens	
	package main

	import (
		"fmt"
		"github.com/dvsekhvalnov/jose2go"
	)

	func main() {

		payload :=  `{"hello": "world"}`
	
		token,err := jose.Sign(payload,jose.NONE, nil)

		if(err==nil) {
			//go use token
			fmt.Printf("\nPlaintext = %v\n",token)
		}
	}

### Creating signed tokens
#### HS-256, HS-384 and HS-512
Signing with HS256, HS384, HS512 expecting `[]byte` array key of corresponding length:

	package main

	import (
		"fmt"
		"github.com/dvsekhvalnov/jose2go"
	)

	func main() {

		payload :=  `{"hello": "world"}`
	
		key := []byte{97,48,97,50,97,98,100,56,45,54,49,54,50,45,52,49,99,51,45,56,51,100,54,45,49,99,102,53,53,57,98,52,54,97,102,99}		
	
		token,err := jose.Sign(payload,jose.HS256,key)

		if(err==nil) {
			//go use token
			fmt.Printf("\nHS256 = %v\n",token)
		}
	}
	
#### RS-256, RS-384 and RS-512, PS-256, PS-384 and PS-512
Signing with RS256, RS384, RS512, PS256, PS384, PS512 expecting `*rsa.PrivateKey` private key of corresponding length. **jose2go** [provides convinient utils](#dealing-with-keys) to construct `*rsa.PrivateKey` instance from PEM encoded PKCS1 or PKCS8 data: `Rsa.ReadPrivate([]byte)` under `jose2go/keys/rsa` package.

	package main

	import (
		"fmt"
		"io/ioutil"
		"github.com/dvsekhvalnov/jose2go/keys/rsa"
		"github.com/dvsekhvalnov/jose2go"
	)

	func main() {

		payload :=  `{"hello": "world"}`

		keyBytes,err := ioutil.ReadFile("private.key")

		if(err!=nil) {
			panic("invalid key file")
		}

		privateKey,e:=Rsa.ReadPrivate(keyBytes)

		if(e!=nil) {
			panic("invalid key format")
		}
	
		token,err := jose.Sign(payload,jose.RS256, privateKey)

		if(err==nil) {
			//go use token
			fmt.Printf("\nRS256 = %v\n",token)
		}
	}	

#### ES-256, ES-384 and ES-512
ES256, ES384, ES512 ECDSA signatures expecting `*ecdsa.PrivateKey` private elliptic curve key of corresponding length.  **jose2go** [provides convinient utils](#dealing-with-keys) to construct `*ecdsa.PrivateKey` instance from PEM encoded PKCS1 or PKCS8 data: `ecc.ReadPrivate([]byte)` or directly from `X,Y,D` parameters: `ecc.NewPrivate(x,y,d []byte)` under `jose2go/keys/ecc` package.

	package main

	import (
	    "fmt"
	    "github.com/dvsekhvalnov/jose2go/keys/ecc"
	    "github.com/dvsekhvalnov/jose2go"
	)

	func main() {

	    payload := `{"hello":"world"}`

		privateKey:=ecc.NewPrivate([]byte{4, 114, 29, 223, 58, 3, 191, 170, 67, 128, 229, 33, 242, 178, 157, 150, 133, 25, 209, 139, 166, 69, 55, 26, 84, 48, 169, 165, 67, 232, 98, 9},
		 			 			   []byte{131, 116, 8, 14, 22, 150, 18, 75, 24, 181, 159, 78, 90, 51, 71, 159, 214, 186, 250, 47, 207, 246, 142, 127, 54, 183, 72, 72, 253, 21, 88, 53},
								   []byte{ 42, 148, 231, 48, 225, 196, 166, 201, 23, 190, 229, 199, 20, 39, 226, 70, 209, 148, 29, 70, 125, 14, 174, 66, 9, 198, 80, 251, 95, 107, 98, 206 })
	
	    token,err := jose.Sign(payload, jose.ES256, privateKey)

	    if(err==nil) {
	        //go use token
	        fmt.Printf("\ntoken = %v\n",token)
	    }
	}  

### Creating encrypted tokens
#### RSA-OAEP-256, RSA-OAEP and RSA1\_5 key management algorithm
RSA-OAEP-256, RSA-OAEP and RSA1_5 key management expecting `*rsa.PublicKey` public key of corresponding length.

	package main

	import (
	    "fmt"
		"io/ioutil"
	    "github.com/dvsekhvalnov/jose2go/keys/rsa"
	    "github.com/dvsekhvalnov/jose2go"
	)

	func main() {

		payload :=  `{"hello": "world"}`
	
		keyBytes,err := ioutil.ReadFile("public.key")

		if(err!=nil) {
			panic("invalid key file")
		}

		publicKey,e:=Rsa.ReadPublic(keyBytes)

		if(e!=nil) {
			panic("invalid key format")
		}

		//OR:
		//token,err := jose.Encrypt(payload, jose.RSA1_5, jose.A256GCM, publicKey)		
		token,err := jose.Encrypt(payload, jose.RSA_OAEP, jose.A256GCM, publicKey)

	    if(err==nil) {
	        //go use token
	        fmt.Printf("\ntoken = %v\n",token)
	    }
	}  
#### AES Key Wrap key management family of algorithms
AES128KW, AES192KW and AES256KW key management requires `[]byte` array key of corresponding length

	package main

	import (
		"fmt"
		"github.com/dvsekhvalnov/jose2go"
	)

	func main() {

		payload :=  `{"hello": "world"}`

		sharedKey :=[]byte{194,164,235,6,138,248,171,239,24,216,11,22,137,199,215,133}

		token,err := jose.Encrypt(payload,jose.A128KW,jose.A128GCM,sharedKey)

		if(err==nil) {
			//go use token
			fmt.Printf("\nA128KW A128GCM = %v\n",token)
		}
	}

#### AES GCM Key Wrap key management family of algorithms
AES128GCMKW, AES192GCMKW and AES256GCMKW key management requires `[]byte` array key of corresponding length

	package main

	import (
		"fmt"
		"github.com/dvsekhvalnov/jose2go"
	)

	func main() {

		payload :=  `{"hello": "world"}`

		sharedKey :=[]byte{194,164,235,6,138,248,171,239,24,216,11,22,137,199,215,133}

		token,err := jose.Encrypt(payload,jose.A128GCMKW,jose.A128GCM,sharedKey)

		if(err==nil) {
			//go use token
			fmt.Printf("\nA128GCMKW A128GCM = %v\n",token)
		}
	}

#### ECDH-ES and ECDH-ES with AES Key Wrap key management family of algorithms
ECDH-ES and ECDH-ES+A128KW, ECDH-ES+A192KW, ECDH-ES+A256KW key management requires `*ecdsa.PublicKey` elliptic curve key of corresponding length. **jose2go** [provides convinient utils](#dealing-with-keys) to construct `*ecdsa.PublicKey` instance from PEM encoded PKCS1 X509 certificate or PKIX data: `ecc.ReadPublic([]byte)` or directly from `X,Y` parameters: `ecc.NewPublic(x,y []byte)`under `jose2go/keys/ecc` package:

	package main

	import (
	    "fmt"
	    "github.com/dvsekhvalnov/jose2go/keys/ecc"
	    "github.com/dvsekhvalnov/jose2go"
	)

	func main() {

	    payload := `{"hello":"world"}`

	    publicKey:=ecc.NewPublic([]byte{4, 114, 29, 223, 58, 3, 191, 170, 67, 128, 229, 33, 242, 178, 157, 150, 133, 25, 209, 139, 166, 69, 55, 26, 84, 48, 169, 165, 67, 232, 98, 9},
	                             []byte{131, 116, 8, 14, 22, 150, 18, 75, 24, 181, 159, 78, 90, 51, 71, 159, 214, 186, 250, 47, 207, 246, 142, 127, 54, 183, 72, 72, 253, 21, 88, 53})

	    token,err := jose.Encrypt(payload, jose.ECDH_ES, jose.A128CBC_HS256, publicKey)

	    if(err==nil) {
	        //go use token
	        fmt.Printf("\ntoken = %v\n",token)
	    }
	}  


#### PBES2 using HMAC SHA with AES Key Wrap key management family of algorithms
PBES2-HS256+A128KW, PBES2-HS384+A192KW, PBES2-HS512+A256KW key management requires `string` passphrase from which actual key will be derived 

	package main

	import (
		"fmt"
		"github.com/dvsekhvalnov/jose2go"
	)

	func main() {

		payload :=  `{"hello": "world"}`

		passphrase := `top secret`

		token,err := jose.Encrypt(payload,jose.PBES2_HS256_A128KW,jose.A256GCM,passphrase)

		if(err==nil) {
			//go use token
			fmt.Printf("\nPBES2_HS256_A128KW A256GCM = %v\n",token)
		}
	}

#### DIR direct pre-shared symmetric key management
Direct key management with pre-shared symmetric keys expecting `[]byte` array key of corresponding length:

	package main

	import (
		"fmt"
		"github.com/dvsekhvalnov/jose2go"
	)

	func main() {

		payload :=  `{"hello": "world"}`
	
		sharedKey :=[]byte{194,164,235,6,138,248,171,239,24,216,11,22,137,199,215,133}
	
		token,err := jose.Encrypt(payload,jose.DIR,jose.A128GCM,sharedKey)

		if(err==nil) {
			//go use token
			fmt.Printf("\nDIR A128GCM = %v\n",token)
		}
	}
	
	
### Creating compressed & encrypted tokens
#### DEFLATE compression
**jose2go** supports optional DEFLATE compression of payload before encrypting, can be used with all supported encryption and key management algorithms:

	package main

	import (
		"fmt"
		"github.com/dvsekhvalnov/jose2go"
	)

	func main() {

		payload :=  `{"hello": "world"}`

		sharedKey :=[]byte{194,164,235,6,138,248,171,239,24,216,11,22,137,199,215,133}

		token,err := jose.Compress(payload,jose.DIR,jose.A128GCM,jose.DEF, sharedKey)

		if(err==nil) {
			//go use token
			fmt.Printf("\nDIR A128GCM DEFLATED= %v\n",token)
		}
	}

	
### Verifying, Decoding and Decompressing tokens
Decoding json web tokens is fully symmetric to creating signed or encrypted tokens (with respect to public/private cryptography), decompressing deflated payloads is handled automatically:

**HS256, HS384, HS512** signatures, **A128KW, A192KW, A256KW**,**A128GCMKW, A192GCMKW, A256GCMKW** and **DIR** key management algorithm expecting `[]byte` array key:

	package main

	import (
		"fmt"
		"github.com/dvsekhvalnov/jose2go"
	)

	func main() {

		token := "eyJhbGciOiJIUzI1NiIsImN0eSI6InRleHRcL3BsYWluIn0.eyJoZWxsbyI6ICJ3b3JsZCJ9.chIoYWrQMA8XL5nFz6oLDJyvgHk2KA4BrFGrKymjC8E"
	
		sharedKey :=[]byte{97,48,97,50,97,98,100,56,45,54,49,54,50,45,52,49,99,51,45,56,51,100,54,45,49,99,102,53,53,57,98,52,54,97,102,99}
	
		payload,err := jose.Decode(token,sharedKey)

		if(err==nil) {
			//go use token
			fmt.Printf("\npayload = %v\n",payload)
		}
	}

**RS256, RS384, RS512**,**PS256, PS384, PS512** signatures expecting `*rsa.PublicKey` public key of corresponding length. **jose2go** [provides convinient utils](#dealing-with-keys) to construct `*rsa.PublicKey` instance from PEM encoded PKCS1 X509 certificate or PKIX data: `Rsa.ReadPublic([]byte)` under `jose2go/keys/rsa` package:

	package main

	import (
	    "fmt"
	    "io/ioutil"
	    "github.com/dvsekhvalnov/jose2go/keys/rsa"
	    "github.com/dvsekhvalnov/jose2go"
	)

	func main() {

	    token := "eyJhbGciOiJSUzI1NiIsImN0eSI6InRleHRcL3BsYWluIn0.eyJoZWxsbyI6ICJ3b3JsZCJ9.NL_dfVpZkhNn4bZpCyMq5TmnXbT4yiyecuB6Kax_lV8Yq2dG8wLfea-T4UKnrjLOwxlbwLwuKzffWcnWv3LVAWfeBxhGTa0c4_0TX_wzLnsgLuU6s9M2GBkAIuSMHY6UTFumJlEeRBeiqZNrlqvmAzQ9ppJHfWWkW4stcgLCLMAZbTqvRSppC1SMxnvPXnZSWn_Fk_q3oGKWw6Nf0-j-aOhK0S0Lcr0PV69ZE4xBYM9PUS1MpMe2zF5J3Tqlc1VBcJ94fjDj1F7y8twmMT3H1PI9RozO-21R0SiXZ_a93fxhE_l_dj5drgOek7jUN9uBDjkXUwJPAyp9YPehrjyLdw"

	    keyBytes,err := ioutil.ReadFile("public.key")

	    if(err!=nil) {
	        panic("invalid key file")
	    }

	    publicKey,e:=Rsa.ReadPublic(keyBytes)

	    if(e!=nil) {
	        panic("invalid key format")
	    }
	
	    payload,err := jose.Decode(token, publicKey)

	    if(err==nil) {
	        //go use token
	        fmt.Printf("\npayload = %v\n",payload)
	    }
	}  

**RSA-OAEP-256**, **RSA-OAEP** and **RSA1_5** key management algorithms expecting `*rsa.PrivateKey` private key of corresponding length:

	package main

	import (
	    "fmt"
	    "io/ioutil"
	    "github.com/dvsekhvalnov/jose2go/keys/rsa"
	    "github.com/dvsekhvalnov/jose2go"
	)

	func main() {

	    token := "eyJhbGciOiJSU0ExXzUiLCJlbmMiOiJBMjU2R0NNIn0.ixD3WVOkvaxeLKi0kyVqTzM6W2EW25SHHYCAr9473Xq528xSK0AVux6kUtv7QMkQKgkMvO8X4VdvonyGkDZTK2jgYUiI06dz7I1sjWJIbyNVrANbBsmBiwikwB-9DLEaKuM85Lwu6gnzbOF6B9R0428ckxmITCPDrzMaXwYZHh46FiSg9djChUTex0pHGhNDiEIgaINpsmqsOFX1L2Y7KM2ZR7wtpR3kidMV3JlxHdKheiPKnDx_eNcdoE-eogPbRGFdkhEE8Dyass1ZSxt4fP27NwsIer5pc0b922_3XWdi1r1TL_fLvGktHLvt6HK6IruXFHpU4x5Z2gTXWxEIog.zzTNmovBowdX2_hi.QSPSgXn0w25ugvzmu2TnhePn.0I3B9BE064HFNP2E0I7M9g"

	    keyBytes,err := ioutil.ReadFile("private.key")

	    if(err!=nil) {
	        panic("invalid key file")
	    }

	    privateKey,e:=Rsa.ReadPrivate(keyBytes)

	    if(e!=nil) {
	        panic("invalid key format")
	    }

	    payload,err := jose.Decode(token, privateKey)

	    if(err==nil) {
	        //go use payload
	        fmt.Printf("\npayload = %v\n",payload)
	    }
	}  

**PBES2-HS256+A128KW, PBES2-HS384+A192KW, PBES2-HS512+A256KW** key management algorithms expects `string` passpharase as a key

	package main

	import (
		"fmt"
		"github.com/dvsekhvalnov/jose2go"
	)

	func main() {

		token :=  `eyJhbGciOiJQQkVTMi1IUzI1NitBMTI4S1ciLCJlbmMiOiJBMjU2R0NNIiwicDJjIjo4MTkyLCJwMnMiOiJlZWpFZTF0YmJVbU5XV2s2In0.J2HTgltxH3p7A2zDgQWpZPgA2CHTSnDmMhlZWeSOMoZ0YvhphCeg-w.FzYG5AOptknu7jsG.L8jAxfxZhDNIqb0T96YWoznQ.yNeOfQWUbm8KuDGZ_5lL_g`

		passphrase := `top secret`

		payload,err := jose.Decode(token,passphrase)

		if(err==nil) {
			//go use token
			fmt.Printf("\npayload = %v\n",payload)
		}
	}

**ES256, ES284, ES512** signatures expecting `*ecdsa.PublicKey` public elliptic curve key of corresponding length. **jose2go** [provides convinient utils](#dealing-with-keys) to construct `*ecdsa.PublicKey` instance from PEM encoded PKCS1 X509 certificate or PKIX data: `ecc.ReadPublic([]byte)` or directly from `X,Y` parameters: `ecc.NewPublic(x,y []byte)`under `jose2go/keys/ecc` package:

	package main

	import (
	    "fmt"
	    "github.com/dvsekhvalnov/jose2go/keys/ecc"
	    "github.com/dvsekhvalnov/jose2go"
	)

	func main() {

	    token := "eyJhbGciOiJFUzI1NiIsImN0eSI6InRleHRcL3BsYWluIn0.eyJoZWxsbyI6ICJ3b3JsZCJ9.EVnmDMlz-oi05AQzts-R3aqWvaBlwVZddWkmaaHyMx5Phb2NSLgyI0kccpgjjAyo1S5KCB3LIMPfmxCX_obMKA"

		publicKey:=ecc.NewPublic([]byte{4, 114, 29, 223, 58, 3, 191, 170, 67, 128, 229, 33, 242, 178, 157, 150, 133, 25, 209, 139, 166, 69, 55, 26, 84, 48, 169, 165, 67, 232, 98, 9},
		 			 			 []byte{131, 116, 8, 14, 22, 150, 18, 75, 24, 181, 159, 78, 90, 51, 71, 159, 214, 186, 250, 47, 207, 246, 142, 127, 54, 183, 72, 72, 253, 21, 88, 53})
	
	    payload,err := jose.Decode(token, publicKey)

	    if(err==nil) {
	        //go use token
	        fmt.Printf("\npayload = %v\n",payload)
	    }
	}
	
**ECDH-ES** and **ECDH-ES+A128KW**, **ECDH-ES+A192KW**, **ECDH-ES+A256KW** key management expecting `*ecdsa.PrivateKey` private elliptic curve key of corresponding length.  **jose2go** [provides convinient utils](#dealing-with-keys) to construct `*ecdsa.PrivateKey` instance from PEM encoded PKCS1 or PKCS8 data: `ecc.ReadPrivate([]byte)` or directly from `X,Y,D` parameters: `ecc.NewPrivate(x,y,d []byte)` under `jose2go/keys/ecc` package:

	package main

	import (
	    "fmt"
	    "github.com/dvsekhvalnov/jose2go/keys/ecc"
	    "github.com/dvsekhvalnov/jose2go"
	)

	func main() {

	    token := "eyJhbGciOiJFQ0RILUVTIiwiZW5jIjoiQTEyOENCQy1IUzI1NiIsImVwayI6eyJrdHkiOiJFQyIsIngiOiItVk1LTG5NeW9IVHRGUlpGNnFXNndkRm5BN21KQkdiNzk4V3FVMFV3QVhZIiwieSI6ImhQQWNReTgzVS01Qjl1U21xbnNXcFZzbHVoZGJSZE1nbnZ0cGdmNVhXTjgiLCJjcnYiOiJQLTI1NiJ9fQ..UA3N2j-TbYKKD361AxlXUA.XxFur_nY1GauVp5W_KO2DEHfof5s7kUwvOgghiNNNmnB4Vxj5j8VRS8vMOb51nYy2wqmBb2gBf1IHDcKZdACkCOMqMIcpBvhyqbuKiZPLHiilwSgVV6ubIV88X0vK0C8ZPe5lEyRudbgFjdlTnf8TmsvuAsdtPn9dXwDjUR23bD2ocp8UGAV0lKqKzpAw528vTfD0gwMG8gt_op8yZAxqqLLljMuZdTnjofAfsW2Rq3Z6GyLUlxR51DAUlQKi6UpsKMJoXTrm1Jw8sXBHpsRqA.UHCYOtnqk4SfhAknCnymaQ"

		privateKey:=ecc.NewPrivate([]byte{4, 114, 29, 223, 58, 3, 191, 170, 67, 128, 229, 33, 242, 178, 157, 150, 133, 25, 209, 139, 166, 69, 55, 26, 84, 48, 169, 165, 67, 232, 98, 9},
		 			 			   []byte{131, 116, 8, 14, 22, 150, 18, 75, 24, 181, 159, 78, 90, 51, 71, 159, 214, 186, 250, 47, 207, 246, 142, 127, 54, 183, 72, 72, 253, 21, 88, 53},
								   []byte{ 42, 148, 231, 48, 225, 196, 166, 201, 23, 190, 229, 199, 20, 39, 226, 70, 209, 148, 29, 70, 125, 14, 174, 66, 9, 198, 80, 251, 95, 107, 98, 206 })

	    payload,err := jose.Decode(token, privateKey)

	    if(err==nil) {
	        //go use token
	        fmt.Printf("\npayload = %v\n",payload)
	    }
	}	
	
### Dealing with keys	
**jose2go** provides several helper methods to simplify loading & importing of elliptic and rsa keys. Import `jose2go/keys/rsa` or `jose2go/keys/ecc` respectively: 

#### RSA keys
1. `Rsa.ReadPrivate(raw []byte) (key *rsa.PrivateKey,err error)` attempts to parse RSA private key from PKCS1 or PKCS8 format (`BEGIN RSA PRIVATE KEY` and `BEGIN PRIVATE KEY` headers)

		package main

		import (
			"fmt"
		    "github.com/dvsekhvalnov/jose2go/keys/rsa"
			"io/ioutil"
		)

		func main() {
	
		    keyBytes,_ := ioutil.ReadFile("private.key")

		    privateKey,err:=Rsa.ReadPrivate(keyBytes)

		    if(err!=nil) {
		        panic("invalid key format")
		    }
	
			fmt.Printf("privateKey = %v\n",privateKey)
		}


2. `Rsa.ReadPublic(raw []byte) (key *rsa.PublicKey,err error)` attempts to parse RSA public key from PKIX key format or PKCS1 X509 certificate (`BEGIN PUBLIC KEY` and `BEGIN CERTIFICATE` headers)
 
		package main

		import (
			"fmt"
		    "github.com/dvsekhvalnov/jose2go/keys/rsa"
			"io/ioutil"
		)

		func main() {
	
		    keyBytes,_ := ioutil.ReadFile("public.cer")

		    publicKey,err:=Rsa.ReadPublic(keyBytes)

		    if(err!=nil) {
		        panic("invalid key format")
		    }
	
			fmt.Printf("publicKey = %v\n",publicKey)
		}
 
#### ECC keys
1. `ecc.ReadPrivate(raw []byte) (key *ecdsa.PrivateKey,err error)` attemps to parse elliptic curve private key from PKCS1 or PKCS8 format (`BEGIN EC PRIVATE KEY` and `BEGIN PRIVATE KEY` headers)

		package main

		import (
			"fmt"
		    "github.com/dvsekhvalnov/jose2go/keys/ecc"
			"io/ioutil"
		)

		func main() {

		    keyBytes,_ := ioutil.ReadFile("ec-private.pem")

		    ecPrivKey,err:=ecc.ReadPrivate(keyBytes)

		    if(err!=nil) {
		        panic("invalid key format")
		    }

			fmt.Printf("ecPrivKey = %v\n",ecPrivKey)
		}

2. `ecc.ReadPublic(raw []byte) (key *ecdsa.PublicKey,err error)` attemps to parse elliptic curve public key from PKCS1 X509 or PKIX format (`BEGIN PUBLIC KEY` and `BEGIN CERTIFICATE` headers)

		package main

		import (
			"fmt"
		    "github.com/dvsekhvalnov/jose2go/keys/ecc"
			"io/ioutil"
		)

		func main() {

		    keyBytes,_ := ioutil.ReadFile("ec-public.key")

		    ecPubKey,err:=ecc.ReadPublic(keyBytes)

		    if(err!=nil) {
		        panic("invalid key format")
		    }

			fmt.Printf("ecPubKey = %v\n",ecPubKey)
		}

3. `ecc.NewPublic(x,y []byte) (*ecdsa.PublicKey)` constructs elliptic public key from (X,Y) represented as bytes. Supported are NIST curves P-256,P-384 and P-521. Curve detected automatically by input length.

		package main

		import (
			"fmt"
		    "github.com/dvsekhvalnov/jose2go/keys/ecc"
		)

		func main() {

		    ecPubKey:=ecc.NewPublic([]byte{4, 114, 29, 223, 58, 3, 191, 170, 67, 128, 229, 33, 242, 178, 157, 150, 133, 25, 209, 139, 166, 69, 55, 26, 84, 48, 169, 165, 67, 232, 98, 9},
				 				    []byte{131, 116, 8, 14, 22, 150, 18, 75, 24, 181, 159, 78, 90, 51, 71, 159, 214, 186, 250, 47, 207, 246, 142, 127, 54, 183, 72, 72, 253, 21, 88, 53})

			fmt.Printf("ecPubKey = %v\n",ecPubKey)
		}

4. `ecc.NewPrivate(x,y,d []byte) (*ecdsa.PrivateKey)` constructs elliptic private key from (X,Y) and D represented as bytes. Supported are NIST curves P-256,P-384 and P-521. Curve detected automatically by input length.

		package main

		import (
			"fmt"
		    "github.com/dvsekhvalnov/jose2go/keys/ecc"
		)

		func main() {

		    ecPrivKey:=ecc.NewPrivate([]byte{4, 114, 29, 223, 58, 3, 191, 170, 67, 128, 229, 33, 242, 178, 157, 150, 133, 25, 209, 139, 166, 69, 55, 26, 84, 48, 169, 165, 67, 232, 98, 9},
				 					  []byte{131, 116, 8, 14, 22, 150, 18, 75, 24, 181, 159, 78, 90, 51, 71, 159, 214, 186, 250, 47, 207, 246, 142, 127, 54, 183, 72, 72, 253, 21, 88, 53},
									  []byte{ 42, 148, 231, 48, 225, 196, 166, 201, 23, 190, 229, 199, 20, 39, 226, 70, 209, 148, 29, 70, 125, 14, 174, 66, 9, 198, 80, 251, 95, 107, 98, 206 })

			fmt.Printf("ecPrivKey = %v\n",ecPrivKey)
		}

### More examples
Checkout `jose_test.go` for more examples.	