# Hardening Builds

Given all sources including dependency definitions are trusted, we can move on to the next step --- build. Let's look back at [../02-towards-trustful-chains](../02-towards-trustful-chains):

> Even if we succeed in depending on only trustworthy artifacts, we still need to ensure what we're depending on are _what we decided to trust at the point in time_. At least we need to:
>
> - **Define dependencies statically to lock them (like `go.sum`)**
> - **Use locked dependencies following the definition properly**
>
> Another challenge is that compiled binaries or containers often lack information on what it is depending on internally, resulting in operation challenges. Suppose that you see a news that a researcher found that `package-blah-blah` had included malwares. Once you forget what kind of tools and packages container images depends on, you'll give a big sigh and lose time. This is one of the reasons why we need to have Software Bill of Materials (SBoM).

In this chapter, you'll try to build the following while generating SBoMs:

- Exercise 1: a Go app container image
- Exercise 2: an Alpine-based container image

## Preliminaries

Run the following commands to install requirements:

```sh
go install github.com/google/ko@v0.11.2
```

## Exercise 1: Build a Container Image with a Go Application Using `ko`

### Build

Run the following to build a Go image:

```sh
$ pushd ./ko-src; KO_DOCKER_REPO=ttl.sh ko build ./; popd

2022/08/11 09:04:10 No matching credentials were found, falling back on anonymous
2022/08/11 09:04:11 Using base gcr.io/distroless/static:nonroot@sha256:59d91a17dbdd8b785e61da81c9095b78099cad8d7757cc108f49e4fb564ef8b3 for example-app
2022/08/11 09:04:12 Building example-app for linux/amd64
2022/08/11 09:04:13 Publishing ttl.sh/example-app-22674c31fceab15d3ac4fb665ad6cc0f:latest
2022/08/11 09:04:18 pushed blob: sha256:22b18b6743847a8831d8d6f095a2ec44c61235f8250ac3c9bf35dd0b46c770cd
2022/08/11 09:04:18 pushed blob: sha256:7040c798bde084f106c5a2544c0c1994263b44c01202a22dfba3200e8a495227
2022/08/11 09:04:20 ttl.sh/example-app-22674c31fceab15d3ac4fb665ad6cc0f:sha256-12742d1638a6fe621a9a35b69e9d074a1883ea4aa34d62e48dfb187e92ac5388.sbom: digest: sha256:570c92ce9c5f2e1d7cb4c81bbd6e5e223018d54361c0a07caac1840af348f161 size: 368
2022/08/11 09:04:20 Published SBOM ttl.sh/example-app-22674c31fceab15d3ac4fb665ad6cc0f:sha256-12742d1638a6fe621a9a35b69e9d074a1883ea4aa34d62e48dfb187e92ac5388.sbom
2022/08/11 09:04:21 existing blob: sha256:250c06f7c38e52dc77e5c7586c3e40280dc7ff9bb9007c396e06d96736cf8542
2022/08/11 09:04:21 existing blob: sha256:b9f88661235d25835ef747dab426861d51c4e9923b92623d422d7ac58eb123e9
2022/08/11 09:04:25 pushed blob: sha256:215b1be207df3beaec417ca216a64cab240aa02f2d78f72f82fca616548fc181
2022/08/11 09:04:28 pushed blob: sha256:44d2ddf44b284d08e27df9724e65247715f8b6d6029f18906b397560b4aef0fe
2022/08/11 09:04:29 ttl.sh/example-app-22674c31fceab15d3ac4fb665ad6cc0f:latest: digest: sha256:12742d1638a6fe621a9a35b69e9d074a1883ea4aa34d62e48dfb187e92ac5388 size: 750
2022/08/11 09:04:29 Published ttl.sh/example-app-22674c31fceab15d3ac4fb665ad6cc0f@sha256:12742d1638a6fe621a9a35b69e9d074a1883ea4aa34d62e48dfb187e92ac5388
ttl.sh/example-app-22674c31fceab15d3ac4fb665ad6cc0f@sha256:12742d1638a6fe621a9a35b69e9d074a1883ea4aa34d62e48dfb187e92ac5388
```

### Review SBoM

`ko` generates a SBoM with a container image and push it to the same OCI registry. You'll see the following line in the logs above:

```sh
2022/08/11 09:04:20 ttl.sh/<redacted>.sbom: digest: <snipped>
```

Let's check what it includes. Run the following command after replacing `<redacted>`:

```sh
crane blob ttl.sh/<redacted>.sbom@$(crane manifest ttl.sh/<redacted>.sbom | jq -r ".layers[0].digest")
```

You'll get the following logs:

```
SPDXVersion: SPDX-2.2
DataLicense: CC0-1.0
SPDXID: SPDXRef-DOCUMENT
DocumentName: example-app
DocumentNamespace: http://spdx.org/spdxpackages/example-app
Creator: Tool: ko v0.11.2
Created: 1970-01-01T00:00:00Z

##### Package representing example-app

PackageName: example-app
SPDXID: SPDXRef-Package-example-app
PackageSupplier: Organization: example-app
PackageDownloadLocation: https://example-app
FilesAnalyzed: false
PackageHomePage: https://example-app
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
PackageLicenseComments: NOASSERTION
PackageComment: NOASSERTION

Relationship: SPDXRef-DOCUMENT DESCRIBES SPDXRef-Package-example-app


Relationship: SPDXRef-Package-example-app DEPENDS_ON SPDXRef-Package-github.com.fatih.color-v1.13.0

##### Package representing github.com/fatih/color

PackageName: github.com/fatih/color
SPDXID: SPDXRef-Package-github.com.fatih.color-v1.13.0
PackageVersion: v1.13.0
PackageSupplier: Organization: github.com/fatih/color
PackageDownloadLocation: https://proxy.golang.org/github.com/fatih/color/@v/v1.13.0.zip
FilesAnalyzed: false
PackageChecksum: SHA256: f0b3987352983cf9b228cb8df105760cd45635b2e8e8b67488bb3cfa6947e77c
PackageLicenseConcluded: NOASSERTION
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION
PackageLicenseComments: NOASSERTION
PackageComment: NOASSERTION
Relationship: SPDXRef-Package-example-app DEPENDS_ON SPDXRef-Package-github.com.mattn.go-colorable-v0.1.9

<snip...>
```

This data follows [SPDX](https://spdx.dev/) format, which has been promoted by Linux Foundation.

## Exercise 2: Build an Alpine-based Container Image Using `apko`

`apko` is a tool to build and publish OCI container images built from APK packages. For instance, the following commands generate a container image with SBoM:

```sh
# in fish
set IMAGE_NAME ttl.sh/(uuidgen | tr [:upper:] [:lower:]):4h
docker run -v "$PWD":/work distroless.dev/apko build apko-src/alpine-base.yaml $IMAGE_NAME apko-alpine.tar
crane push apko-alpine.tar $IMAGE_NAME

# in bash
IMAGE_NAME=ttl.sh/$(uuidgen | tr [:upper:] [:lower:]):4h
docker run -v "$PWD":/work distroless.dev/apko build apko-src/alpine-base.yaml $IMAGE_NAME apko-alpine.tar
docker load < apko-alpine.tar
```

After `apko` exit successfully, you'll find some files like `sbom-*` in your current directory. Check them out to see what they include.

## Exercise (Optional): Review other build tools

See other build tools like [Buildpacks](https://buildpacks.io/), [Bazel](https://bazel.build/), [Syft](https://github.com/anchore/syft), [Trivy](https://github.com/aquasecurity/trivy), [FOSSology](https://www.fossology.org/), [OSS Review Toolkit](https://github.com/oss-review-toolkit/ort), etc.
