# Spring Security

authentication and access-control framework (and protection against common exploits.).

Spring Security provides protection against common exploits. Whenever possible, the protection is enabled by default.

Spring Security provides integrations with numerous frameworks and APIs (Cryptography, Spring Data, Java’s Concurrency APIs, Jackson, Localization).

## Spring Boot with Maven

<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-security</artifactId>
</dependency>

## Password Storage

Spring Security’s PasswordEncoder interface is used to perform a one way transformation of a password to allow the password to be stored securely.

## DelegatingPasswordEncoder

Prior to Spring Security 5.0 the default PasswordEncoder was NoOpPasswordEncoder which required plain text passwords.
Spring Security introduces DelegatingPasswordEncoder which solves problems by:
- Ensuring that passwords are encoded using the current password storage recommendations (There are many applications using old password encodings that cannot easily migrate);
- Allowing for validating passwords in modern and legacy formats (The best practice for password storage will change again);
- Allowing for upgrading the encoding in the future (As a framework Spring Security cannot make breaking changes frequently);
You can easily construct an instance of DelegatingPasswordEncoder using PasswordEncoderFactories.
PasswordEncoder passwordEncoder = PasswordEncoderFactories.createDelegatingPasswordEncoder();
Alternatively, you may create your own custom instance.

## Password Storage Format

The general format for a password is: {id}encodedPassword
Such that id is an identifier used to look up which PasswordEncoder should be used and encodedPassword is the original encoded password for the selected PasswordEncoder.

## Password Matching

Matching is done based upon the {id} and the mapping of the id to the PasswordEncoder provided in the constructor.
By default, the result of invoking matches(CharSequence, String) with a password and an id that is not mapped (including a null id) will result in an IllegalArgumentException. This behavior can be customized using DelegatingPasswordEncoder.setDefaultPasswordEncoderForMatches(PasswordEncoder).

## Getting Started Experience

User user = User.withDefaultPasswordEncoder()
  .username("user")
  .password("password")
  .roles("user")
  .build();
System.out.println(user.getPassword());
// {bcrypt}$2a$10$dXJ3SW6G7P50lGmMkkmwe.20cQQubK3.HZWzG3YB1tlRy.fqvM/BG

### BCryptPasswordEncoder

The BCryptPasswordEncoder implementation uses the widely supported bcrypt algorithm to hash the passwords. In order to make it more resistent to password cracking, bcrypt is deliberately slow.

### Argon2PasswordEncoder

The Argon2PasswordEncoder implementation uses the Argon2 algorithm to hash the passwords. Argon2 is the winner of the Password Hashing Competition. In order to defeat password cracking on custom hardware, Argon2 is a deliberately slow algorithm that requires large amounts of memory.

### Pbkdf2PasswordEncoder, SCryptPasswordEncoder

The Pbkdf2PasswordEncoder implementation uses the PBKDF2 algorithm to hash the passwords. In order to defeat password cracking PBKDF2 is a deliberately slow algorithm. Like other adaptive one-way functions, it should be tuned to take about 1 second to verify a password on your system.

The SCryptPasswordEncoder implementation uses scrypt algorithm to hash the passwords. In order to defeat password cracking on custom hardware scrypt is a deliberately slow algorithm that requires large amounts of memory. Like other adaptive one-way functions, it should be tuned to take about 1 second to verify a password on your system.

### Other PasswordEncoders

There are a significant number of other PasswordEncoder implementations that exist entirely for backward compatibility.

## Change Password Configuration

A Well-Known URL for Changing Passwords indicates a mechanism by which password managers can discover the password update endpoint for a given application.
You can configure Spring Security to provide this discovery endpoint. For example, if the change password endpoint in your application is /change-password, then you can configure Spring Security like so:
	http.passwordManagement(Customizer.withDefaults())
Then, when a password manager navigates to /.well-known/change-password then Spring Security will redirect your endpoint, /change-password.
Or, if your endpoint is something other than /change-password, you can also specify that like so:
http.passwordManagement((management) -> management
	.changePasswordPage("/update-password")
)
With the above configuration, when a password manager navigates to /.well-known/change-password, then Spring Security will redirect to /update-password.

https://docs.spring.io/spring-security/reference/features/authentication/password-storage.html ................

https://spring.io/projects/spring-security