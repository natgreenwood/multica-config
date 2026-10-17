BRANCH NAMES
- Trunk branches are used as-is, with no prefix or description: `main`, `master`, `develop`.
- All other branches use the format `<type>/<description>`.
- Types:
  - `feat`: new functionality
  - `fix`: non-urgent bug fixes
  - `chore`: maintenance, dependencies, tooling
  - `release`: release preparation
- Description rules:
  - Lowercase letters (a-z) and digits (0-9) only.
  - Words are separated by a single hyphen (-).
  - Dots (.) are allowed inside a word, for example in version numbers like 2.4.1. A dot must have a letter or digit on both sides.
  - Exactly one `/`, between the type and the description.
  - No uppercase, underscores, spaces or other special characters. No consecutive hyphens or dots.
  - No hyphens or dots at the start or end of the description.

SECRETS
- Bitwarden Secrets Manager (`bws`) is installed on the runtime. Use it to retrieve secrets as needed.
