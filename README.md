# OpenEBS Mayastor multi-arch image builder

This is a github workflow that builds and uploads the required images
to run mayastor for both amd64 & arm64.

It doesn't touch the upstream code as I'm not into maintaining code,
especially of the openebs caliber. I'm using the exact same prcedures
that openebs follow to build the images - I just do that on arm runners
as well and merge the resulting images to a multi-arch image

> [!NOTE]  
> A minor patch is applied during the build of the mayastor repo on the arm runner
>
> For more see https://github.com/openebs/openebs/issues/3789#issuecomment-3056414733

## Usage

In order to be able to run openebs/mayastor, a couple of helm values
are needed to achieve the following:
- Remove amd64 nodeSelectors
- Use the generated images instead of the upstream ones
- Replace some images that are not multi-arch (e.g. etcd)

Required values for the openebs helm chart:

```yaml
nodeSelector: {}
image:
  registry: ghcr.io/dzervas

csi:
  image:
    registry: ghcr.io/dzervas

mayastor:
  nodeSelector: {}
  image:
    registry: ghcr.io/dzervas
  io_engine:
    nodeSelector:
      # Keep only the engine nodeSelector and get rid of the arch
      "openebs.io/engine": mayastor
  csi:
    node:
      initContainers:
        enabled: false

  etcd:
    image:
      # The 3.5.6 image used by the etcd version in the mayastor helm chart
      # is not available for arm64
      tag: 3.5-debian-12
    volumePermissions:
      image:
        repository: bitnami/os-shell
```
