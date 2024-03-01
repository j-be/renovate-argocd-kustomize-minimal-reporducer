# Minimal reproducer for Kustomize image support in ArgoCD Application manifests

This is a minimal reproducer requested as part of [discussion#23848](https://github.com/renovatebot/renovate/discussions/23848).

## Current behavior as of Renovate 37.221.1

The entry in [application-kustomize.yaml](application-kustomize.yaml#L16) `spec.source.kustomize.images` is not updated even though there is a newer version for this image (see [here](https://github.com/j-be/desec-dns-operator/pkgs/container/desec-dns-operator)).

## Expected behavior

Renovate updates `ghcr.io/j-be/desec-dns-operator` to its most recent version.

## Steps to reproduce

### Requirements

* `npm`

### Steps

* Clone this repo
* `npm install renovate`
* `LOG_LEVEL=DEBUG npx renovate --platform=local`

The log should now contain something like:

```
DEBUG: packageFiles with updates (repository=local)
       "config": {
         "argocd": [
          {
             "deps": [
               {
                 "depName": "ghcr.io/j-be/desec-dns-operator",
                 "currentValue": "0.1.7",
                 "datasource": "docker",
                 "updates": [
                   {
                     "bucket": "non-major",
                     "newVersion": "0.1.9",
                     "newValue": "0.1.9",
                     "newMajor": 0,
                     "newMinor": 1,
                     "updateType": "patch",
                     "branchName": "renovate/ghcr.io-j-be-desec-dns-operator-0.x"
                   }
                 ],
               }
             ],
           }
         ]
       }
```

## Known workaround

This can currently be achieved by adding the following to `renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  ...
  "customManagers": [
    {
      "customType": "regex",
      "fileMatch": [
        ".*"
      ],
      "matchStrings": [
        "\\s+-\\s+.*=(?<depName>.+):(?<currentValue>.+)"
      ],
      "datasourceTemplate": "docker"
    }
  ]
}
```

Note however, that this does not cover special cases, like e.g. quotes.
