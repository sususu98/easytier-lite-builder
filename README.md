# easytier-lite-builder

GitHub Actions builder for minimal, **non-UPX** static musl `easytier-core` binaries targeting low-RAM embedded routers (e.g. 256 MiB RAM, no swap, `aarch64-unknown-linux-musl`).

## Why No-UPX + Feature Trimming?

Official `easytier-core` releases are full-featured (~22 MiB uncompressed) and packed with UPX (~5.6 MiB). On Linux routers without swap, UPX decompresses the entire executable into non-reclaimable anonymous RAM (`RssAnon`), pinning ~22 MiB of physical memory permanently.

By disabling unused features (QUIC, WebSocket, WireGuard portal, Magic DNS, KCP, SOCKS5, FakeTCP, mimalloc/jemalloc, OpenSSL) and compiling with `opt-level = "z"` without UPX:
- Code pages stay file-backed (`RssFile`), demand-paged, and reclaimable by the Linux kernel.
- Binary size fits directly in small flash partitions while cutting `RssAnon` substantially.

## Default Features (`v2.6.4`)

```
--no-default-features --features "aes-gcm,tun,zstd"
```

- `aes-gcm`: Required to stay compatible with default AES-GCM mesh encryption without pulling in `wireguard`/`ring` or vendored OpenSSL.
- `tun`: Enables kernel TUN interface (`et0`) and kernel-backed TCP proxy (`KernelTcpStream`), avoiding `smoltcp`.
- `zstd`: Keeps compatibility with peers that enable `data_compress_algo = 2` (Zstd).

## Trigger Build

```bash
gh workflow run build.yml \
  -f easytier_ref=v2.6.4 \
  -f features=aes-gcm,tun,zstd \
  -f target=aarch64-unknown-linux-musl \
  -f opt_level=z \
  -f publish_release=true
```
