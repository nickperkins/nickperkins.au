---
title: "ORAS for Configuration Management"
date: 2024-06-29T18:00:00+10:00
draft: false

categories: [platform engineering]
tags: [ORAS, OCI, configuration management, containers]
toc: false
author: "Nick Perkins"
---
A challenge for any engineering team is handling configuration management in a secure and efficient way. Recently, I've explored using ORAS (OCI Registry As Storage), a tool which enhances OCI (Ope Container Initiative) registries by enabling them to store various artifacts, not just container images.

## What is ORAS?

[ORAS](https://oras.land) is an open source project which builds tools and libraries to enable using OCI registries to store any type of artifact, not just container images. The ability to leverage container registries in this way comes from how container images are constructed.

## Container Image Basics

A container image is not a single file but is instead made up of layers. Just like a cake, each layer builds on top of the next layer to form the complete container image. Each layer contains the data that forms the image, which is stored as a blob. Another blob contains the configuration for the container image. Finally, a manifest brings all of this together to form the full container image. Below is the manifest for an alpine linux image.

```json
{
   "schemaVersion": 2,
   "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
   "config": {
      "mediaType": "application/vnd.docker.container.image.v1+json",
      "size": 1483,
      "digest": "sha256:011cd18df9954a4143ac1256824ad34026d382d26714a2b222d0a8f06286224f"
   },
   "layers": [
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "size": 3367154,
         "digest": "sha256:3d2af5f613c84e549fb09710d45b152d3cdf48eb7a37dc3e9c01e2b3975f4f76"
      }
   ]
}
  ```

## OCI Artifacts and ORAS

It was always possible to piece together your own 'container image' that wasn't really an image, but was a way to store other information in a container registry. In 2019, OCI approved a new project to look at standardising the way that artifacts were stored in OCI registries. This culminated with the release of v1.1.0 of the OCI Image and OCI Distribution specifications.

ORAS is the "de facto tool" for working with OCI artifacts. It allows you to not only push and pull artifacts to and from a container registry, but also attach artifacts to existing artifacts, including container images.

## Pushing and Pulling Artifacts with ORAS

It's very simple to push and pull an artifact to a container registry with ORAS. As a demonstration, we'll start with a simple json file and push it to a local test container registry.

```shell
$ oras push localhost:5001/configs/simpleconfig:v1 config.json
✓ Uploaded  config.json                                  22/22  B 100.00%   24ms
  └─ sha256:3a3a3af8c336832376065d8138563bc288faa5ac1f33c17e66aa57a2a6e60451
✓ Uploaded  application/vnd.oci.empty.v1+json              2/2  B 100.00%   25ms
  └─ sha256:44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a
✓ Uploaded  application/vnd.oci.image.manifest.v1+json 590/590  B 100.00%     0s
  └─ sha256:6bc7f143e75a7d5791e182ed80eadd242e62492b5aa5a541ec6842bde01b5296
Pushed [registry] localhost:5001/configs/simpleconfig:v1
ArtifactType: application/vnd.unknown.artifact.v1
Digest: sha256:6bc7f143e75a7d5791e182ed80eadd242e62492b5aa5a541ec6842bde01b5296
```

You can see that the blob is uploaded, along with the manifest and configuration. Let's look at the manifest for this artifact.

```shell
$ oras manifest fetch localhost:5001/configs/simpleconfig:v1 --pretty
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  "artifactType": "application/vnd.unknown.artifact.v1",
  "config": {
    "mediaType": "application/vnd.oci.empty.v1+json",
    "digest": "sha256:44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a",
    "size": 2,
    "data": "e30="
  },
  "layers": [
    {
      "mediaType": "application/vnd.oci.image.layer.v1.tar",
      "digest": "sha256:3a3a3af8c336832376065d8138563bc288faa5ac1f33c17e66aa57a2a6e60451",
      "size": 22,
      "annotations": {
        "org.opencontainers.image.title": "config.json"
      }
    }
  ],
  "annotations": {
    "org.opencontainers.image.created": "2024-06-29T06:46:22Z"
  }
}
```

As you can see, it's very similar to a container image, with a config created by ORAS and the blob containing your artifact as a layer. ORAS makes it simple to pull that artifact back down too.

```shell
$ oras pull localhost:5001/configs/simpleconfig:v1 -o output
✓ Pulled      config.json                                22/22  B 100.00%    3ms
  └─ sha256:3a3a3af8c336832376065d8138563bc288faa5ac1f33c17e66aa57a2a6e60451
✓ Pulled      application/vnd.oci.image.manifest.v1+j. 568/568  B 100.00%   83µs
  └─ sha256:7abc32780ea52f9ca2d21a3c6e5aaeaa0773df17a7bb3a0150b3fe657b2790fc
Pulled [registry] localhost:5001/configs/simpleconfig:v1
Digest: sha256:7abc32780ea52f9ca2d21a3c6e5aaeaa0773df17a7bb3a0150b3fe657b2790fc
```

This has pull down our artifact and placed it in the output directory. Neat!

```shell
$ ls output/
config.json
```

## Attaching Artifacts to Container Images

What if your configuration relates to a container image that you will be deploying? Wouldn't it make sense to have them linked somehow, making it easier to get the right config file for the right image on deployment? ORAS makes that easy with the ability to attach artifacts to an existing container image.

Let's push a container image up to demonstrate this. Since the test registry I am using is only able to accept OCI images, I'm using skopeo to push my image that I've exported with `docker save`.

```shell
$ skopeo copy --dest-tls-verify=false --format=oci docker-archive:testimage.tar docker://localhost:5001/testimage:v1
Getting image source signatures
Copying blob 11b9733114f7 done   |
Copying blob 48205ccc81c1 done   |
Copying blob 74f83e71d791 done   |
Copying blob 9cb81ccf927f done   |
Copying blob 72c690143394 done   |
Copying blob 70632f097359 done   |
Copying blob fbc1665d4d36 done   |
Copying config 007bc662c1 done   |
Writing manifest to image destination
```

Now I can attach my configuration to the image in the registry.

```shell
$ oras attach --artifact-type=custom/config localhost:5001/testimage:v1 config.json
✓ Exists    application/vnd.oci.empty.v1+json              2/2  B 100.00%     0s
  └─ sha256:44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a
✓ Exists    config.json                                  22/22  B 100.00%     0s
  └─ sha256:3a3a3af8c336832376065d8138563bc288faa5ac1f33c17e66aa57a2a6e60451
✓ Uploaded  application/vnd.oci.image.manifest.v1+json 732/732  B 100.00%   18ms
  └─ sha256:9db7a89f5a639e35d1efd1f6f132ade14c3bda8858b7427ee9fb0f65bb82b97c
Attached to [registry] localhost:5001/testimage@sha256:ba0a3a59071528a8f02293e62a839a2cb90973a67a21b34f80508334b969ddcf
Digest: sha256:9db7a89f5a639e35d1efd1f6f132ade14c3bda8858b7427ee9fb0f65bb82b97c
```

You will see that because the blob already exists in the registry, it has reused that as part of our configuration. This is similar to how layers can be shared between container images.

Now that the configuration is attached, how do we see this and then access the artifact later? We can discover which artifacts refer to an image.

```shell
$ oras discover localhost:5001/testimage:v1
localhost:5001/testimage@sha256:ba0a3a59071528a8f02293e62a839a2cb90973a67a21b34f80508334b969ddcf
└── custom/config
    └── sha256:9db7a89f5a639e35d1efd1f6f132ade14c3bda8858b7427ee9fb0f65bb82b97c
```

You'll see that we have the image and then a `custom/config` artifact and the related sha. We can easily pull down the artifact now as before.

```shell
$ oras pull localhost:5001/testimage@sha256:9db7a89f5a639e35d1efd1f6f132ade14c3bda8858b7427ee9fb0f65bb82b97c -o output
✓ Pulled      config.json                                22/22  B 100.00%  724µs
  └─ sha256:3a3a3af8c336832376065d8138563bc288faa5ac1f33c17e66aa57a2a6e60451
✓ Pulled      application/vnd.oci.image.manifest.v1+j. 732/732  B 100.00%  134µs
  └─ sha256:9db7a89f5a639e35d1efd1f6f132ade14c3bda8858b7427ee9fb0f65bb82b97c
Pulled [registry] localhost:5001/testimage@sha256:9db7a89f5a639e35d1efd1f6f132ade14c3bda8858b7427ee9fb0f65bb82b97c
Digest: sha256:9db7a89f5a639e35d1efd1f6f132ade14c3bda8858b7427ee9fb0f65bb82b97c
$ ls output/
config.json
```

## Summary

Storing artifacts with your container images using ORAS makes it simple to distribute configuration fils in the same way that you distribute your container images. The ability to then link these artifacts to your container images makes it simple to know exactly which configuration version to use when deploying your software. This is great in a situation where your ops team needs to run a rollback late at night - no need to consult separate documentation or raise the on call software engineer from their slumber.
