# Installing Internal Gems

To be able to install internal gems, the environment variable `BUNDLE_RUBYGEMS__PKG__GITHUB__COM` needs to be set.

## Create the Token

1. Navigate to [New personal access token](https://github.com/settings/tokens/new) in GitHub
2. Set the expiration.
3. Check the `read:packages` permission.
4. Click "Generate Token"

## Add the Token to Your Environment

Add the token to your profile such as `.bashrc`, `.zshrc`, or `~/.config/fish/config.fish`:

For bash and zsh:

```bash
export BUNDLE_RUBYGEMS__PKG__GITHUB__COM=token
```

For fish:

```fish
set -x BUNDLE_RUBYGEMS__PKG__GITHUB__COM token
```
