Create an IAM Role.

This role uses Boto to create the IAM Role. You will also need to use a `Principal` (account) with the `Allow` permission to `iam:CreateRole`.
This is an elevated permission. You will need to run this role with appropriate credentials.

If using AWS Credentials file (`~/.aws/credentials`) and Profile (e.g. `[admin]` section), you can use `--extra-var "profile=admin"` so that Boto will use the proper credentials.

