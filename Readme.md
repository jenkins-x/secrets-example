# Test project to help figure out working with secrets and kubernetes pipelines

## Prerequisits

### helm
```sh
brew install kubernetes-helm
```
### helm-secrets
```sh
helm plugin install https://github.com/futuresimple/helm-secrets
```
### gpg
```sh
brew install gpg
gpg --gen-key
```
Take note of the gpg fingerprint key, for this test example we have `2E3DB643E57A3A0CE87C19CF656F6D8AC99D96EE`.

Add `export GPG_TTY=$(tty)` to your .bash or .zshrc profile
```
echo 'export GPG_TTY=$(tty)' >> ~/.zshrc
source ~/.zshrc
```
```sh
gpg --export-secret-keys >~/.gnupg/secring.gpg
```
## Usage
Create a new local git project
```sh
mkdir secrets-example
cd secrets-example
git init
```
Create a `.sops.yaml` file with the fingerprint key above included, for example:
```yaml
creation_rules:
    - pgp: "2E3DB643E57A3A0CE87C19CF656F6D8AC99D96EE"
```
Create or add to your `.gitignore` file so we don't accidentily commit decoded secrets later
```
*.dec
```
Create a `secrets.yaml` and add a secret to it:
```yaml
my-password: foo
```

Encrypt your secrets:
```sh
helm secrets enc secrets.yaml
```
Now check your `secrets.yaml`, my-password value should be encrypted.

Lets round trip and make sure we can decode our new secret, __if you get an error__ at this next stage check the [FAQ](#faq) below
```sh
helm secrets dec secrets.yaml
```
Enter your gpg passphrase and you will have a `secrets.yaml.dec` created
```sh
cat secrets.yaml.dec
```
Files can now be committed and pushed to a public repo providing you added the `.dec` extension to the project .gitignore.

## FAQ

### ERROR: Could not load secring: open ~/.gnupg/secring.gpg: no such file or

Run gpg --export-secret-keys >~/.gnupg/secring.gpg

### ERROR: Inappropriate ioctl for device

Add `export GPG_TTY=$(tty)` to your .bash or .zshrc profile
```
echo 'export GPG_TTY=$(tty)' >> ~/.zshrc
source ~/.zshrc
```