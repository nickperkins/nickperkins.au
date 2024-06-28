---
title: "ORAS for Configuration Management"
date: 2024-06-28T15:21:39+10:00
draft: true

categories: []
tags: []
toc: false
author: "Nick Perkins"
---
A challenge for any engineering team is handling configuration management in a secure and efficient way. Recently, I've explored using ORAS (OCI Registry As Storage), a tool which enhances OCI (Ope Container Initiative) registries by enabling them to store various artifacts, not just container images. This approach offers some interesting ideas that could help teams with this challenge.

## Container Image Basics

It is helpful to have a general understanding of what a container image is made up of, and how they are stored, to understand how ORAS can do what it does.

A container image is not a single file but is instead made up of layers. Just like a cake, each layer builds on top of the next layer to form the complete container image. Each layer contains the data that forms the image, which is stored as compressed blobs. Another blob contains the configuration for the container image. Finally, a manifest brings all these pieces together to form the full container image. Below is the manifest for an alpine linux image.

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

## Everything is a Blob

If a container image is simply a bunch of blobs with a manifest to manage it, why can’t we store other blobs in a container registry? Well, the answer is you can. This is how ORAS can store generic artifacts in a OCI registry.