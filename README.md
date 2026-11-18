# pmmp-yolks

Pterodactyl images for PocketMine-MP and its forks: `ghcr.io/superhub-host/pmmp-yolks:<tag>`.

| Tag | PHP runtime | For |
|---|---|---|
| `pm4-php8.1` | pmmp/PHP-Binaries | PocketMine-MP 4 |
| `pm5-php8.1`, `pm5-php8.2`, `pm5-php8.3`, `pm5-php8.4` | pmmp/PHP-Binaries | PocketMine-MP 5 and forks that run on stock PM5 binaries |
| `amber-php8.4` | Amber-PM/PHP-Binaries (`amber-php-8.4.27-r5`, PHP 8.4 ZTS + `ext-rtc`) | AmberPM 1.7+ (NetherNet, Bedrock 26.60+) |

pmmp/PHP-Binaries is archived, but its releases are still downloadable, so the `pm*` tags are built the same way as before.

## Adding a runtime for another fork

Add an entry to the matrix in `.github/workflows/build.yml`:

```yaml
- tag: <fork>-php<version>
  binaries_repo: <owner>/PHP-Binaries      # GitHub repo with the release
  binaries_tag: <release tag>
  binaries_file: <Linux x86_64 .tar.gz asset>
  binaries_checksums: SHA256SUMS.txt       # optional, verified if set
  required_extensions: <ext> <ext>         # optional, build fails if not loaded
```

The archive must have the pmmp layout (`bin/php7/bin/php`, `bin/php7/lib/php/extensions/*-zts-*`). The runtime is installed to `/usr/php` and linked as `php`.

## AmberPM and NetherNet

Bedrock 26.60 clients only connect over NetherNet. Besides the `amber-php8.4` image, the server needs:

- `AMBER_NETHERNET_PUBLIC_IP`: the node's public IP.
- `AMBER_NETHERNET_PORT`: an extra allocation (UDP) for NetherNet game traffic. If it is empty, AmberPM uses the main port and turns RakNet off, so only 26.50+ clients can join.
- A backup of `nethernet.key` (the server's NetherNet identity).
