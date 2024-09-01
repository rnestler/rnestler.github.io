Title: Switching from Docker to Podman
Tags: Linux, ArchLinux, Docker, Podman
Status: draft
Language: en
Summary: Explaining my reasons to switch from Docker to Podman and some troubles I encountered.

# Notes

 * podman is rootless
 * docker: Either sudo or being in the docker group, which is a security risk
 * Trick to avoid being in the docker group: group password and newgrp


## Issues

### No default registry
```
Error: short-name "outofcoffee/imposter-all:3.36.3" did not resolve to an alias and no unqualified-search registries are defined in "/etc/containers/registries.conf"
```

 -> `docker.io/outofcoffee/imposter-all:3.36.3`

### User mapping

```
ERRO[0000] cannot find UID/GID for user raphael: no subuid ranges found for user "raphael" in /etc/subuid - check rootless mode in man pages.
WARN[0000] Using rootless single mapping into the namespace. This might break some images. Check /etc/subuid and /etc/subgid for adding sub*ids if not using a network user
```

See <https://wiki.archlinux.org/title/Podman#Set_subuid_and_subgid>

```
sudo usermod --add-subuids 100000-165535 --add-subgids 100000-165535 raphael
podman system migrate
```

