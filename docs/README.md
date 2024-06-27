# OpSecId Labs


## Setup

### Clone the repo
```
git clone https://github.com/OpSecId/labs.git
```

### Launch agent and create a did
```bash
# Launch agent
cd labs
docker compose -f ./docker/docker-compose.yml up

# Create did
curl -X 'POST' \
  'http://localhost:8020/wallet/did/create' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "method": "key",
  "options": {
    "key_type": "ed25519"
  },
  "seed": "00000000000000000000000000000000"
}' | jq .result.did

```

### Issue a credential
```bash
VC=$(curl -X 'POST' \
  'http://localhost:8020/vc/credentials/issue' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "credential": {
    "@context": [
      "https://www.w3.org/2018/credentials/v1",
      "https://w3id.org/vdl/v2"
    ],
    "type": ["VerifiableCredential", "Iso18013DriversLicenseCredential"],
    "issuer": "did:key:z6MkgKA7yrw5kYSiDuQFcye4bMaJpcfHFry3Bx45pdWh3s8i",
    "issuanceDate": "2023-11-15T10:00:00-07:00",
    "credentialSubject": {
      "type": "LicensedDriver",
      "driversLicense": {
        "type": "Iso18013DriversLicense",
        "age_over_18": true,
        "family_name": "TURNER",
        "given_name": "SUSAN",
        "issue_date": "2023-01-15T10:00:00-07:00",
        "expiry_date": "2028-08-27T12:00:00-06:00"
      }
    }
  },
  "options": {}
}
' | jq .verifiableCredential)

```

### Verify the credential

#### With a third party application
Navigate to the [Univerifier](https://univerifier.io/) and paste the content of the VC into the input field.

#### With the agent
```bash
curl -X 'POST' \
  'http://localhost:8020/vc/credentials/verify' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{ 
  "verifiableCredential": {
  "@context": [
    "https://www.w3.org/2018/credentials/v1",
    "https://w3id.org/vdl/v2",
    "https://w3id.org/security/suites/ed25519-2020/v1"
  ],
  "type": [
    "VerifiableCredential",
    "Iso18013DriversLicenseCredential"
  ],
  "issuer": "did:key:z6MkgKA7yrw5kYSiDuQFcye4bMaJpcfHFry3Bx45pdWh3s8i",
  "issuanceDate": "2023-11-15T10:00:00-07:00",
  "credentialSubject": {
    "type": "LicensedDriver",
    "driversLicense": {
      "type": "Iso18013DriversLicense",
      "age_over_18": true,
      "family_name": "TURNER",
      "given_name": "SUSAN",
      "issue_date": "2023-01-15T10:00:00-07:00",
      "expiry_date": "2028-08-27T12:00:00-06:00"
    }
  },
  "proof": {
    "type": "Ed25519Signature2020",
    "proofPurpose": "assertionMethod",
    "verificationMethod": "did:key:z6MkgKA7yrw5kYSiDuQFcye4bMaJpcfHFry3Bx45pdWh3s8i#z6MkgKA7yrw5kYSiDuQFcye4bMaJpcfHFry3Bx45pdWh3s8i",
    "created": "2024-06-27T16:27:30+00:00",
    "proofValue": "z32ZYJTZjQKX5Kbnzbays9Puoh4XPeHq4Hs2rXm7c11UwbBJWpVLYpJCUVNJunUXHGE1UE4ndZqvdQbGS7FZ4rmsm"
  }
},
  "options": {}
  }' | jq .verified


```
### Modify the context:
Key swap attack
```json
"@context": [
"https://www.w3.org/2018/credentials/v1",
"https://labs.opsec.id/ctx/vdl/fake_age.jsonld",
"https://w3id.org/security/suites/ed25519-2020/v1"
]
```

Value swap attack
```json
"@context": [
"https://www.w3.org/2018/credentials/v1",
"https://labs.opsec.id/ctx/vdl/swap_names.jsonld",
"https://w3id.org/security/suites/ed25519-2020/v1"
]
```



### A simpler explanation
Unprotected vocab
```
{
    "@context": [
        "https://www.w3.org/2018/credentials/v1",
        "https://www.w3.org/ns/credentials/examples/v2"
    ],
    "type": [
        "VerifiableCredential"
    ],
    "issuer": "did:key:z6MkgKA7yrw5kYSiDuQFcye4bMaJpcfHFry3Bx45pdWh3s8i",
    "issuanceDate": "2024-01-01T00:00:00Z",
    "credentialSubject": {
        "type": "Person",
        "givenName": "Patrick",    
        "familyName": "St-Louis"
    }
}
```
```
{
    "@context": [
      "https://www.w3.org/2018/credentials/v1",
      "https://www.w3.org/ns/credentials/examples/v2",
      {"prénom":"https://www.w3.org/ns/credentials/examples#givenName", "nomDeFamille":"https://www.w3.org/ns/credentials/examples#familyName"}
    ],
    "type": [
      "VerifiableCredential"
    ],
    "issuer": "did:key:z6MkgKA7yrw5kYSiDuQFcye4bMaJpcfHFry3Bx45pdWh3s8i",
    "issuanceDate": "2024-01-01T00:00:00Z",
    "credentialSubject": {
      "type": "Person",
      "prénom": "Patrick",    
      "nomDeFamille": "St-Louis"
    }
}
```