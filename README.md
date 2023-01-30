# buildkit-repro

To reproduce:

``` bash
# create builder
docker buildx create --name bk-test1 --driver-opt "image=moby/buildkit:v0.11.2" --node bk-test1 &&  docker buildx inspect --bootstrap bk-test1 &&  docker buildx use bk-test1

# do a loop 3x just to be able to see multiple image changes; running twice will show it though
for LOOP in {1..3}
do
  # build base
  docker buildx build --provenance=true --builder=bk-test1 --pull --push --progress plain --cache-to=type=inline,mode=max -t mbentley/buildkit-repro:alpine -f Dockerfile.base --cache-from=type=registry,ref=mbentley/buildkit-repro:alpine .

  # build curl image
  docker buildx build --provenance=true --builder=bk-test1 --pull --push --progress plain --cache-to=type=inline,mode=max -t mbentley/buildkit-repro:curl -f Dockerfile.curl --cache-from=type=registry,ref=mbentley/buildkit-repro:curl .

  # pull the image
  docker pull mbentley/buildkit-repro:curl

  # show the image
  docker images | grep mbentley/buildkit-repro
done
```

The image ID will have changed despite the layers not changing - seems to be due to the manifest changing.

``` bash
$ diff <(docker inspect 0afb6413da9b) <(docker inspect cc7de484da6e)
3,6c3,4
<         "Id": "sha256:0afb6413da9b409e2a88c469c117c9e2f6f6d7d7ab2f1a0597f647e9751e5edf",
<         "RepoTags": [
<             "mbentley/buildkit-repro:curl"
<         ],
---
>         "Id": "sha256:cc7de484da6e4152c16abe9b7da85cc7bb41720173fea9dfcd8d677f11570d65",
>         "RepoTags": [],
8c6
<             "mbentley/buildkit-repro@sha256:de1b2fe8cdfbf88b4ebb7cd2256b1c56002143dedbc316c30e51b00dad62c468"
---
>             "mbentley/buildkit-repro@sha256:11dbb23b6a7acc6be628ea3923a1270b60565e021571191ffd95cdc978307c1e"
```
