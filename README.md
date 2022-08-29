# gcp-docker

GitHub Action for building a docker container and pushing it to a GCP artifact
registry.

NOTE: This repo is public so that the GitHub action defined here can be
reused across GitHub projects. Reusing actions defined in private repos
requires GitHub Enterprise.

## Required inputs

* `gcp-sa-creds-json` — GCP service account credentials in JSON
* `name` — image name (e.g. `my-app`)
* `project` — GCP project (e.g. `myorg-eng-0ac3fb`)
* `registry` — GCP registry domain (e.g. `us-central1-docker.pkg.dev`) 
* `repo` — artifact repository name (e.g. `myorg-docker`)
* `tag_branch` — if set to a value other than "none", tag the image with the truncated branch name
* `tag_latest` —  if set to "branch" tag with `${branch}-latest`; if set to "true" or "latest" tag with `latest`

## Example

```yaml
  image_build:
    runs-on: ubuntu-22.04
    needs: test
    steps:
      - name: build and push docker container
        uses: CalthorpeAnalytics/gcp-docker@v0.2.1
        with:
          gcp-sa-creds-json: ${{ secrets.GCP_ENG_DOCKER_SERVICE_ACCOUNT }}
          name: myorg_bigapp
          project: myorg-global
          registry: us-central1-docker.pkg.dev
          repo: docker
          tag_branch: "true"
          tag_latest: "branch"
```

If the build was triggered by a branch called `xy-EXP-99999-scary-bug`, on commit
`b7e23ec29af22b0b4e41da31e868d57226121c84`, the following container image and 
tags will be built and/or pushed:

```
us-central1-docker.pkg.dev/myorg-global/docker/myorg_bigapp:b7e23ec
us-central1-docker.pkg.dev/myorg-global/docker/myorg_bigapp:xy-EXP-99999-scary-bug
us-central1-docker.pkg.dev/myorg-global/docker/myorg_bigapp:xy-EXP-99999-scary-bug-latest
```