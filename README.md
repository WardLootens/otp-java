# OTP-Java

![](https://github./BastiaanJansen/OTP-Java/workflows/Build/badge.svg)
![](https://github.com/BastiaanJansen/OTP-Java/workflows/Test/badge.svg)
[![Codacy Badge](https://app.codacy.com/project/badge/Grade/91d3addee9e94a0cad9436601d4a4e1e)](https://www.codacy.com/gh/BastiaanJansen/OTP-Java/dashboard?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=BastiaanJansen/OTP-Java&amp;utm_campaign=Badge_Grade)
![](https://img.shields.io/github/license/BastiaanJansen/OTP-Java)
![](https://img.shields.io/github/issues/BastiaanJansen/OTP-Java)

A small and easy-to-use one-time password generator for Java according to [RFC 4226](https://tools.ietf.org/html/rfc4226) (HOTP) and [RFC 6238](https://tools.ietf.org/html/rfc6238) (TOTP).

## Features
The following features are supported:
1. Generation of secrets
2. Time-based one-time password (TOTP, RFC 6238) generation based on current time, specific time, OTPAuth URI and more for different HMAC algorithms.
3. HMAC-based one-time password (HOTP, RFC 4226) generation based on counter and OTPAuth URI.
4. Verification of one-time passwords
5. Generation of OTP Auth URI's

## Installation
### Maven
```xml
<dependency>
    <groupId>com.github.bastiaanjansen</groupId>
    <artifactId>OTP-Java</artifactId>
    <version>1.0.5</version>
</dependency>
```

Or you can download the source from the [GitHub releases page](https://github.com/BastiaanJansen/OTP-Java/releases).

## Usage
### HOTP (Counter-based one-time passwords)
#### Initialization HOTP instance

```java
String secret = "VV3KOX7UQJ4KYAKOHMZPPH3US4CJIMH6F3ZKNB5C2OOBQ6V2KIYHM27Q";
HOTPGenerator hotp = new HOTPGenerator(secret);
```

The default length of a generated code is six digits. You can change the default:
```java
String secret = "VV3KOX7UQJ4KYAKOHMZPPH3US4CJIMH6F3ZKNB5C2OOBQ6V2KIYHM27Q";
int passwordLength = 6;
HOTPGenerator hotp = new HOTPGenerator(passwordLength, secret);

// It is also possible to create a HOTPGenerator instance based on an OTPAuth URI. When algorithm or digits are not specified, the default values will be used.
new HOTPGenerator(new URI("otpauth://hotp/issuer?secret=ABCDEFGHIJKLMNOP&algorithm=SHA1&digits=6&counter=8237"));
```

Get information about the generator:

```java
String secret = hotp.getSecret(); // VV3KOX7UQJ4KYAKOHMZPPH3US4CJIMH6F3ZKNB5C2OOBQ6V2KIYHM27Q
int passwordLength = hotp.getPasswordLength(); // 6
HMACAlgorithm algorithm = hotp.getAlgorithm(); // HMACAlgorithm.SHA1
```

#### Generation of HOTP code
After creating an instance of the HOTPGenerator class, a code can be generated by using the `generate()` method:
```java
try {
    int counter = 5;
    String code = hotp.generate(counter);
    
    // To verify a token:
    boolean isValid = hotp.verify(code, counter);
    
    // Or verify with a delay window
    boolean isValid = hotp.verify(code, counter, 2);
} catch (IllegalStateException e) {
    // Handle error
}
```

### TOTP (Time-based one-time passwords)
#### Initialization TOTP instance
TOTPGenerator can accept more paramaters: `passwordLength`, `period`, `algorithm` and `secret`. The default values are: passwordLength = 6, period = 30, algorithm = SHA1.

```java
String secret = "VV3KOX7UQJ4KYAKOHMZPPH3US4CJIMH6F3ZKNB5C2OOBQ6V2KIYHM27Q";
int passwordLength = 8; // Password length must be between 6 and 8
Duration period = Duration.ofSeconds(30); // This can of course be any period
HMACAlgorithm algorithm = HMACAlgorithm.SHA1; // SHA256 and SHA512 are also supported

TOTPGenerator totp = new TOTPGenerator(passwordLength, period, algorithm, secret);
```

Get information about the generator:
```java
String secret = totp.getSecret(); // VV3KOX7UQJ4KYAKOHMZPPH3US4CJIMH6F3ZKNB5C2OOBQ6V2KIYHM27Q
int passwordLength = totp.getPasswordLength(); // 6
HMACAlgorithm algorithm = totp.getAlgorithm(); // HMACAlgorithm.SHA1
Duration period = totp.getPeriod(); // Duration.ofSeconds(30)
```

#### Generation of TOTP code
After creating an instance of the TOTPGenerator class, a code can be generated by using the `generate()` method, similarly with the HOTPGenerator class:
```java
try {
    String code = totp.generate();
     
    // To verify a token:
    boolean isValid = totp.verify(code);
} catch (IllegalStateException e) {
    // Handle error
}
```
The above code will generate a time-based one-time password based on the current time. The API supports, besides the current time, the creation of codes based on `timeSince1970` in milliseconds, `Date`, `Instant` and `URI` values:

```java
try {
    // Based on current time
    totp.generate();
    
    // Based on specific date
    totp.generate(new Date());
    
    // Based on seconds past 1970
    totp.generate(9238346823);
    
    // Based on an instant
    totp.generate(Instant.now());
    
    // Based on an OTPAuth URI. When algorithm, period or digits are not specified, the default values will be used
    totp.generate(new URI("otpauth://totp/issuer?secret=ABCDEFGHIJKLMNOP&algorithm=SHA1&digits=6&period=30"));
} catch (IllegalStateException e) {
    // Handle error
}
```

### Generate secrets
You can use your own secrets, or you can generate secrets with `SecretGenerator`:
```java
// To generate a secret with 160 bits
String secret = SecretGenerator.generate();

// To generate a secret with a custom amount of bits
String secret = SecretGenerator.generate(512);
```

### Generation of OTPAuth URI's
To easily generate a OTPAuth URI for easy on-boarding, use the `getURI()` method for both `HOTPGenerator` and `TOTPGenerator`. Example for `TOTPGenerator`:
```java
TOTPGenerator totp = new TOTPGenerator(secret);

URI uri = totp.getURI("issuer", "account"); // otpauth://totp/issuer:account?period=30&digits=6&secret=SECRET&algorithm=SHA1

```

## Licence
OTP-Java is available under the MIT licence. See the LICENCE for more info.
