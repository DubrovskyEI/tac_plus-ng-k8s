##### Building a new image with a tag of certain version to use in the helm template

```Bash
docker build -f -f Dockerfile -t "${DOCKER_REGISTRY}/tac-plus-ng-k8s:51a7dfa .
```

`51a7dfa` - number of the last commit in the repository during the build

##### Create a new chart with name `tac-plus-ng`

```Bash
helm create tac-plus-ng
```

#### Using SOPS and Helm Secrets to storing sensitive data securely

##### Install `helm-secrets` plugin

```Bash
helm plugin install https://github.com/jkroepke/helm-secrets
```

##### Install `SOPS`

Installation of latest version `v3.8.1` following instruction:
https://github.com/getsops/sops/releases

Download the binary

```Bash
curl -LO https://github.com/getsops/sops/releases/download/v3.8.1/sops-v3.8.1.linux.amd64
```

Move the binary into PATH

```Bash
sudo mv sops-v3.8.1.linux.amd64 /usr/local/bin/sops
```

Make the binary executable

```Bash
sudo chmod +x /usr/local/bin/sops
```

##### Generate a GPG key

```Bash
export KEY_NAME="tac-plus-ng-secret-key"
export KEY_COMMENT="Key for tac-plus-ng helm chart"

gpg --batch --full-generate-key <<EOF
%no-protection
Key-Type: 1
Key-Length: 4096
Subkey-Type: 1
Subkey-Length: 4096
Expire-Date: 0
Name-Comment: ${KEY_COMMENT}
Name-Real: ${KEY_NAME}
EOF
```

##### Get the GPG key fingerprint

List the generated GPG keys to get the key ID

```Bash
gpg --list-keys
```

Example output:

```Text
/home/egor/.gnupg/pubring.kbx
-----------------------------
pub   rsa4096 2024-05-31 [SCEA]
      D79941D56AFE74B38AA185AF79D2BA41B404D53E
uid           [ultimate] tac-plus-ng-secret-key (Key for tac-plus-ng helm chart)
sub   rsa4096 2024-05-31 [SEA]
```

GPG key fingerprint in this case `D79941D56AFE74B38AA185AF79D2BA41B404D53E`

##### Configure SOPS to use GPG key for encryption/decryption

At the root Helm chart directory create file `.sops.yaml` with the following content:

```YAML
creation_rules:
  - pgp: D79941D56AFE74B38AA185AF79D2BA41B404D53E
```

> [!NOTE]  
> Don't forget to replace the key fingerprint generated in the previous step

##### Create a secret file and `tacacs-key.yaml.dec`

Create file tacacs-key.yaml.dec with sensitive data in plain text with the following content:

```Text
secretKey: demo
```

##### Encrypt this file using `helm-secrets`

```Bash 
helm secrets encrypt tacacs-key.yaml.dec > tacacs-key.yaml
```

##### Copy `.sops.yaml` into root Helm chart directory

Put `.sops.yaml` file where local key is referenced at the root of Helm chart which make it available for usage (or into root of helm_charts)

As example for tac-plus-ng Helm chart:

```Bash
cp ./helm-secret/.sops.yaml ./helm-chart/tac-plus-ng/
```

##### Copy file with encrypted data into root Helm chart directory

Put encrypted file into Helm chart directory

```Bash
cp ./helm-secret/tacacs-key.yaml ./helm-chart/tac-plus-ng/
```

From now there is not need to store the plain secret file `tacacs-key.yaml.dec`

#### Generate manifest without deploy

```Bash
helm install tac-plus-ng helm/tac-plus-ng/ --values helm/tac-plus-ng/tacacs-key.yaml --dry-run --debug
```
#### Install Helm chart using `helm-secrets` plugin

```Bash
helm secrets upgrade -i tac-plus-ng helm/tac-plus-ng/ --values helm/tac-plus-ng/tacacs-key.yaml
```

#### List all Helm realeases

```Bash
helm ls
```

#### Show the status of named release

```Bash
helm status tac-plus-ng
```

#### Get information about release

Print a human readable collection of information about the notes, hooks, supplied values, and generated manifest file of named release

```Bash
helm get all tac-plus-ng
```

#### Uninstall the release

```Bash
helm uninstall tac-plus-ng
```

##### References

> References
> 
> 1. https://github.com/jkroepke/helm-secrets/wiki/Installation
> 2. https://github.com/getsops/sops?ref=blog.gitguardian.com
> 3. https://blog.gitguardian.com/how-to-handle-secrets-in-helm/
> 4. https://fenyuk.medium.com/helm-for-kubernetes-handling-secrets-with-helm-secrets-plugin-4e31f6f3e306