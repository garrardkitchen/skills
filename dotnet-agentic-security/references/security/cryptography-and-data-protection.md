# Cryptography and ASP.NET Core Data Protection

Use this file when the user asks about encryption, hashing, signatures, HMAC, password storage, random values, key rotation, cookies, protected payloads, or ASP.NET Core Data Protection.

## Must implement

- Do not invent cryptographic protocols.
- Use ASP.NET Core Data Protection for cookies, antiforgery, and short-lived protected app state.
- Use Key Vault or approved key-management systems for long-lived secrets and keys.
- Use password hashing APIs for passwords, not general-purpose hashes.
- Use cryptographically secure random values for tokens and nonces.
- Use fixed-time comparison for MACs/signatures.
- Design for key rotation.

## Data Protection

```csharp
builder.Services.AddDataProtection()
    .SetApplicationName("MyApp");
```

For multi-instance production apps, persist the key ring to approved shared storage and protect keys at rest, for example with Blob Storage plus Key Vault.

## Random token

```csharp
byte[] bytes = RandomNumberGenerator.GetBytes(32);
string token = WebEncoders.Base64UrlEncode(bytes);
```

## HMAC verification

```csharp
static bool VerifySignature(byte[] payload, byte[] key, byte[] providedSignature)
{
    using var hmac = new HMACSHA256(key);
    byte[] computed = hmac.ComputeHash(payload);
    return CryptographicOperations.FixedTimeEquals(computed, providedSignature);
}
```

## Avoid

- MD5, SHA1, or fast unsalted hashes for passwords.
- Custom encryption formats unless reviewed by cryptography specialists.
- Reusing keys across unrelated purposes.
- Logging protected payloads, keys, tokens, or decrypted secrets.
- Using Data Protection as a general-purpose database encryption strategy.

## Sources

- Microsoft Learn .NET cryptography model and secure coding guidelines
- Microsoft Learn ASP.NET Core Data Protection
- OWASP Cryptographic Storage and Password Storage Cheat Sheets
