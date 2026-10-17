# Authentication

`refrakt gen auth` generates a complete authentication system —
registration, login, logout, password hashing, and session management.
It's real code in your project, not a library. You own it and can
modify everything.

## Generate auth

```bash
refrakt gen auth
```

Auth works with both databases. The generator reads `gleam.toml` and
produces code for whichever one your project uses:

- **PostgreSQL** (`--db postgres`) — the repo uses `pog` and takes
  `ctx.db`; the migration uses `SERIAL` and `TIMESTAMPTZ`.
- **SQLite** (`--db sqlite`) — the repo uses `sqlight` and takes
  `ctx.db_path`; the migration uses `INTEGER PRIMARY KEY AUTOINCREMENT`
  and `TEXT` timestamps.

A project created without `--db` has nowhere to store users, so
`gen auth` prints an error and generates nothing.

This creates 9 files:

| File | Purpose |
|------|---------|
| `domain/user.gleam` | User type |
| `domain/auth.gleam` | Password hashing and verification |
| `data/user_repo.gleam` | User database queries |
| `data/migrations/NNN_create_users.sql` | Users table |
| `web/auth_handler.gleam` | Login, register, logout handlers |
| `web/auth_views.gleam` | Login and register forms |
| `web/forms/auth_form.gleam` | Credential validation |
| `web/middleware/auth.gleam` | Session middleware |
| `test/.../auth_handler_test.gleam` | Password hashing tests |

It also adds the `argus` package (Argon2 password hashing) to
`gleam.toml`.

And adds 5 routes to the router:

```
GET     /login              Login page
POST    /login              Submit login
GET     /register           Register page
POST    /register           Submit registration
POST    /logout             Log out
```

## Run the migration

```bash
refrakt migrate
```

Or apply the SQL by hand.

PostgreSQL:
```bash
psql my_app_dev < src/my_app/data/migrations/NNN_create_users.sql
```

SQLite:
```bash
sqlite3 my_app.db < src/my_app/data/migrations/NNN_create_users.sql
```

## How sessions work

Sessions use signed cookies. When a user logs in:

1. The handler verifies the password
2. Sets a `_user_id` cookie (signed, 7-day expiry)
3. Redirects to `/`

When checking auth:

1. The middleware reads the `_user_id` cookie
2. Verifies the signature
3. Looks up the user in the database
4. Passes the user to the handler or redirects to `/login`

## Protecting routes

Use the `require_auth` middleware in any handler:

```gleam
import my_app/web/middleware/auth

pub fn create(req: Request, ctx: Context) -> Response {
  use user <- auth.require_auth(req, ctx)
  // `user` is the authenticated User
  // ... create the post ...
}
```

If there's no valid session, `require_auth` redirects to `/login`.

### Get the current user (optional)

```gleam
import my_app/web/middleware/auth

pub fn index(req: Request, ctx: Context) -> Response {
  let current_user = auth.get_current_user(req, ctx)
  // Result(User, Nil) — Ok if logged in, Error if not
}
```

## Password hashing

The generated `domain/auth.gleam` hashes passwords with **Argon2id**,
using the [`argus`](https://hex.pm/packages/argus) package. `gen auth`
adds it to your `gleam.toml` (it leaves the file alone if `argus` is
already there):

```toml
[dependencies]
argus = ">= 2.0.0 and < 3.0.0"
```

```gleam
import argus
import gleam/result

pub fn hash_password(password: String) -> Result(String, Nil) {
  argus.hasher()
  |> argus.hash(password)
  |> result.map(fn(output) { output.encoded_hash })
  |> result.replace_error(Nil)
}

pub fn verify_password(password: String, hash: String) -> Bool {
  argus.verify(hash, password)
  |> result.unwrap(False)
}
```

- Every hash gets a fresh random salt, so two users with the same
  password get different hashes.
- `argus.hasher()` uses OWASP's recommended Argon2id settings
  (2 iterations, 19 MiB of memory, parallelism 1). Tune them with
  `argus.time_cost`, `argus.memory_cost` and `argus.parallelism`.
- The stored value is a self-describing string such as
  `$argon2id$v=13$m=19456,t=2,p=1$<salt>$<hash>`. It carries the salt and the
  parameters, so changing the settings later doesn't break old hashes.
- `verify_password` compares in constant time and returns `False` for a
  malformed hash.

`hash_password` returns a `Result` because Argon2 can fail (for
example if memory can't be allocated). The register handler turns that
failure into the same "Could not create account" error as a database
failure.

Argus wraps the reference C implementation as a NIF, so building the
project needs a C compiler (`build-essential` on Debian, `build-base`
on Alpine). See [Deployment](deployment.md).

## Form validation

The auth form validates:
- Email is not empty
- Password is not empty
- Password is at least 8 characters (registration only)
- Password confirmation matches (registration only)

Modify `web/forms/auth_form.gleam` to add stricter validation:

```gleam
// Add to decode_register:
|> validate_email_format(email)

fn validate_email_format(errors, email) {
  case string.contains(email, "@") {
    True -> errors
    False -> [#("email", "Must be a valid email"), ..errors]
  }
}
```

## Customization

Since auth is generated code (not a library), you can:

- Add fields to the User type (name, role, avatar)
- Change the session duration
- Add email verification
- Add password reset
- Add OAuth (Google, GitHub)
- Change the login redirect target
- Add rate limiting to login attempts
- Use a different password hashing algorithm

Edit the files directly — they're yours.
