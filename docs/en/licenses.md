# Licenses

## sysfail itself

sysfail is copyrighted by its author (All rights reserved). Redistribution is prohibited. The full terms are in [LICENSE.md](https://github.com/takuho0115/sysfail/blob/main/LICENSE.md) in the repository.

Copyright notices and license texts for third-party software built into the executables are in `THIRD_PARTY_NOTICES.txt`, attached to each release.

## Software in the game images

On first launch, or when you run `sysfail doctor --build`, sysfail pulls these game images from ghcr.io:

- `ghcr.io/takuho0115/sysfail-el9`
- `ghcr.io/takuho0115/sysfail-toolbox`

Both are based on the official Rocky Linux 9 image (`rockylinux/rockylinux:9`) and contain Rocky Linux 9 packages. Some of them are under licenses such as the GPL and LGPL, which require that the source code be made available. Each package is governed by its own license text.

### Where to get the source

The source for these packages is available as source RPMs (`.src.rpm`) published by Rocky Linux.

| Target | Location |
|---|---|
| Current Rocky Linux 9 | `BaseOS/source/tree/Packages/` and `AppStream/source/tree/Packages/` under <https://dl.rockylinux.org/pub/rocky/9/> |
| Earlier minor versions (from 9.0) | The same layout under `9.<n>/` at <https://dl.rockylinux.org/vault/rocky/> |

### List the packages in an image

```bash
docker run --rm --entrypoint rpm ghcr.io/takuho0115/sysfail-el9@<digest> -qa
```

Replace `<digest>` with the value shown by `docker images --digests`. For the toolbox image, use `sysfail-toolbox`. Replacing `-qa` with `-qa --qf '%{SOURCERPM}\n'` in the command above prints the source RPM file names directly.
