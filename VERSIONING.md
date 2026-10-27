# Versioning

Every push to `main` is tagged `vMAJOR.MINOR.PATCH` by `.github/workflows/release.yml`. Consumers pin a tag in `build.rs`.

## Bump rules

The tag is computed from conventional-commit subjects since the previous tag.

| Bump | Trigger | Examples |
|---|---|---|
| MAJOR | `feat!:` / `BREAKING CHANGE:` in the body | removing or renaming a field, rpc, message or enum; changing a field type or number |
| MINOR | `feat:` | new field, new rpc, new message, new enum variant |
| PATCH | anything else | comments, formatting, `reserved` on removed numbers |

## Compatibility rules

- A field number is never reused; removed ones are marked `reserved` with the name.
- The type or number of an existing field never changes.
- Removing anything is a MAJOR bump.
- New fields go at the end, with the next free number.

## Consuming a version

```rust
const ACADEMY_PROTO_VERSION: &str = "v1.0.0";

fn main() {
    println!("cargo:rerun-if-env-changed=ACADEMY_PROTO_VERSION");
    let version = std::env::var("ACADEMY_PROTO_VERSION")
        .unwrap_or_else(|_| ACADEMY_PROTO_VERSION.to_string());
    let url = format!(
        "https://raw.githubusercontent.com/my-prop-trading/academy-proto-files/{version}/proto/"
    );
    ci_utils::sync_and_build_proto_file(&url, "AcademyClientsGrpcService.proto");
}
```

| Value | Builds against |
|---|---|
| tag (`v1.2.0`) | that release |
| branch or commit sha (env `ACADEMY_PROTO_VERSION`) | an unmerged contract |
| `main` in the URL | the latest push, unpinned |
