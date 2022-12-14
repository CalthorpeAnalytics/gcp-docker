# gcp-docker

GitHub Action for building a docker container and pushing it to a GCP artifact
registry.

NOTE: This repo is public so that the GitHub action defined here can be
reused across GitHub projects. Reusing actions defined in private repos
requires GitHub Enterprise.

## Required inputs

* `name` — image name (e.g. `my-app`)
* `project` — GCP project (e.g. `myorg-eng-0ac3fb`)
* `registry` — GCP registry domain (e.g. `us-central1-docker.pkg.dev`) 
* `repo` — artifact repository name (e.g. `myorg-docker`)
* `tag_branch` — if set to a value other than "none", tag the image with the truncated branch name
* `tag_latest` —  if set to "branch" tag with `${branch}-latest`; if set to "true" or "latest" tag with `latest`
* `tag_sha` — if set, use the provided SHA sum for tagging (defaults to GITHUB_SHA, set to "none" to disable)

## Optional inputs

* `gcp-sa-creds-json` — GCP service account credentials in JSON; either this or `gcp-identity-provider` must be set (but not both)
* `gcp-identity-provider` - ID of the Workload Identity Federation provider; either this or `gcp-sa-creds-json` must be set (but not both)
* `gcp-service-account` - The email or unique ID of the service account to use via identity federation; must be set if gcp-identity-provider is set

## Example using permanent credentials

```yaml
  image_build:
    runs-on: ubuntu-22.04
    needs: test
    steps:
      - name: build and push docker container
        uses: CalthorpeAnalytics/gcp-docker@v0.2.2
        with:
          gcp-sa-creds-json: ${{ secrets.SERVICE_ACCOUNT }}
          name: myorg_bigapp
          project: myorg-global
          registry: us-central1-docker.pkg.dev
          repo: docker
          tag_branch: "true"
          tag_latest: branch
          tag_sha: ${{ github.event.pull_request.head.sha }}
```

## Example using identity federation

```yaml
  image_build:
    runs-on: ubuntu-22.04
    needs: test
    steps:
      - name: build and push docker container
        uses: CalthorpeAnalytics/gcp-docker@v0.2.2
        with:
          gcp-identity-provider: projects/123456789012/locations/global/workloadIdentityPools/my-identity-pool/providers/my-provder
          gcp-service-account: my-service-account@myorg-global.iam.gserviceaccount.com
          name: myorg_bigapp
          project: myorg-global
          registry: us-central1-docker.pkg.dev
          repo: docker
          tag_branch: "true"
          tag_latest: branch
          tag_sha: ${{ github.event.pull_request.head.sha }}
```

If the build was triggered by a branch called `xy-XYZA-99999-scary-bug` on commit
`b7e23ec29af22b0b4e41da31e868d57226121c84`, the following container image and 
tags will be built and/or pushed:

```
us-central1-docker.pkg.dev/myorg-global/docker/myorg_bigapp:b7e23ec
us-central1-docker.pkg.dev/myorg-global/docker/myorg_bigapp:xy-XYZA-99999-scary-bug
us-central1-docker.pkg.dev/myorg-global/docker/myorg_bigapp:xy-XYZA-99999-scary-bug-latest
```
