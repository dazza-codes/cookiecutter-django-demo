
## Django Secrets

- [GitHub - HBNetwork/python-decouple: Strict separation of config from code.](https://github.com/HBNetwork/python-decouple)
	- use environment files (`.env`) that are ignored by git
- [Code & secret management best practices - GitGuardian Blog](https://blog.gitguardian.com/secrets-api-management/)
	- [Git Clean, Git Remove file from commit - Cheatsheet - GitGuardian Blog](https://blog.gitguardian.com/rewriting-git-history-cheatsheet/)
- [The Right Way to Store Secrets using Parameter Store | AWS Cloud Operations & Migrations Blog](https://aws.amazon.com/blogs/mt/the-right-way-to-store-secrets-using-parameter-store/)
- [https://github.com/sobolevn/git-secret](https://github.com/sobolevn/git-secret)
	- encrypt secret files in a git repo
	- [A Guide to Git-Secret - DEV Community](https://dev.to/vnjogani/a-guide-to-git-secret-49g3)
- [https://github.com/awslabs/git-secrets](https://github.com/awslabs/git-secrets)
	- prevent secrets from getting into git commits
* use `environ` or `django-environ` - e.g. it is used by cookiecutter-django

### Generate a new SECRET_KEY

```python
from django.core.management.utils import get_random_secret_key
print(get_random_secret_key())
```

### Deploying

When deploying to production servers or CI, inject a secret environment
variable (e.g. through GitHub Secrets) that uses a base64 encoded string and
the python `gnupg` package alongside `dotenv` to load the configuration file.

### Using git-secrets

From [https://github.com/awslabs/git-secrets](https://github.com/awslabs/git-secrets)

```sh
brew install git-secrets
# cd to root of git repo and enable it, e.g.
git secrets --install
git secrets --register-aws
```

### Working with git-secret

- [https://github.com/sobolevn/git-secret](https://github.com/sobolevn/git-secret)
- [https://sobolevn.me/git-secret/](https://sobolevn.me/git-secret/)

#### install

```sh
brew install git-secret gnupg
```

#### setup

Generate a GPG key, try RSA with 4096-bit keys:

```sh
gpg --gen-key
```

Manage shared public keys for collaborators

```sh
mkdir -p public_keys
gpg --import public_keys/* 
gpg --batch --yes -a -o public_keys/`echo $USER`.gpg --export $EMAIL
echo "Finished importing and exporting keys"
```

Setup the git secrets:

```sh
# in the root of the git repository
git secret init
git secret tell $EMAIL
```

The new `.gitsecret` directory can be added to the git repository and pushed to
a remote.  Any private files in it should be in the `.gitignore` file.

The project might already ignore all files in `.env` or `.envs/**` (or similar
paths).  Find those `.gitignore` entries and modify them so they will allow
`*.secret` files.  For example:

```text
.env
!.env.secret
.envs/.*/.*
!.envs/.*/.*.secret
```

Store secrets in `.envs/*` files that are encrypted.  Some environment files are targets for secrets.  The secret files should be in `.gitignore` (but not any `*.secret` files).

```sh
$ echo "django_demo_app/.envs/.*/.*" >> .gitignore
$ echo "!django_demo_app/.envs/.*/.*.secret" >> .gitignore
$ find ./django_demo_app/.envs -type f
./django_demo_app/.envs/.local/.postgres
./django_demo_app/.envs/.local/.django
./django_demo_app/.envs/.production/.postgres
./django_demo_app/.envs/.production/.django
```

Add the files as secrets:

```sh
$ find ./django_demo_app/.envs -type f -exec git secret add {} +
git-secret: 4 item(s) added.
```

Encrypt all these files and remove the originals:

```sh
git secret hide -d
```

Share the secrets with collaborators:

```sh
for i in $(gpg -k | grep -Eo "[^<]+@\S+\.[^>]+"); do
   git secret tell $i  2> /dev/null | true
done
```

Use the secrets:

```sh
git secret reveal
```

