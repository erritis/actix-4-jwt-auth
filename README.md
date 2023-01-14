# Actix 4 compatible JWT authentication

In order to make use of this crate, you can add it to your Cargo.toml

This crate is build with actix-4.

```
actix-4-jwt-auth = "1.0.0"
```

Or when you like to use the latest as found on github:

```
actix-4-jwt-auth = {git = "https://github.com/spectare/actix-4-jwt-auth", branch = "main"}
```

Works with extractors

```rust
    #[derive(Debug, PartialEq, Clone, Serialize, Deserialize)]
    pub struct FoundClaims {
        pub iss: String,
        pub sub: String,
        pub aud: String,
        pub name: String,
        pub email: Option<String>,
        pub email_verified: Option<bool>,
    }

    #[get("/authenticated_user")]
    async fn authenticated_user(user: AuthenticatedUser<FoundClaims>) -> String {
        format!("Welcome {}!", user.claims.name)
    }
```

Is a Actix endpoint URL that extracts the AuthenticatedUser from the JWT based Authorization Bearer header.

You can wire your application like

```rust
      let authority = "https://a.valid.openid-connect.idp/".to_string();

      let oidc = Oidc::new(OidcConfig::Issuer(authority.clone().into())).await.unwrap();

      let biscuit_validator = OidcBiscuitValidator { options: ValidationOptions {
              issuer: Validation::Validate(authority),
              ..ValidationOptions::default()
          }
      };

      HttpServer::new(move || {
        App::new()
                .app_data(oidc.clone())
                .wrap(biscuit_validator.clone())
                // .wrap(OidcBiscuitValidator::default()) //without issuer verification
                .service(authenticated_user),
        })
      .bind("0.0.0.0:8080".to_string())?
      .run()
      .await
```

More documentation is found on [docs.rs](https://docs.rs/actix-4-jwt-auth/0.6.1/actix_4_jwt_auth/)

# Development of this crate

In order to run the integration tests, it is neccesary to run a service that mocks OIDC requests.

```sh
docker run -p8080:8080 -e BIND=0.0.0.0  spectare/oidc-token-test-service:latest
```

This service published a keyset with the openid-configuration and allows you to translate _any_ claimset
into a JWT token to be used in your tests. (So that may be valid, faulty or invalid)

```sh
cargo test
```

Thereafter will call the service to test various types of JWT tokens.
