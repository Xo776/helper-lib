# helper-lib

Personal QX ruleset & utility scripts. Self-hosted for stability.

## Files

| File | Description |
|------|-------------|
| `qx.conf` | QX rewrite rules (import via raw URL) |
| `media.js` | Response processing engine |

## Usage

Import in QX `[rewrite_local]`:

```
https://raw.githubusercontent.com/Xo776/by-yt/main/qx.conf, tag=QX Rules, enabled=true
```

Requires `udp_drop_list=443` in `[general]`.

## Sync

```bash
curl -o media.js https://raw.githubusercontent.com/Maasea/sgmodule/master/Script/Youtube/youtube.response.js
git add media.js qx.conf
git commit -m "sync $(date +%Y-%m-%d)"
git push
```

---

*Last updated: 2026-05-26*
