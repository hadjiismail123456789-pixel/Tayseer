# Tayseer phone updates — free via GitHub (no Capgo Cloud)

The phone app checks this repo on every launch and stages newer bundles itself.
No account, no subscription. Two files per release:

```
tayseer-updates/
  README.md        <- you are here
  latest.json      <- what phones read: version + zip URL + checksum
```

## Publishing a new update (maintainer)

1. Bump `tayseer/package.json` version, e.g. `1.0.1` -> `1.0.2`
2. `cd tayseer` then `npm run build`
3. `npx @capgo/cli bundle zip` -> prints the zip path + sha256 `checksum`
4. Upload the zip to `tayseer-updates/bundles/` in this repo
5. Update `tayseer-updates/latest.json`:
   ```json
   { "version": "1.0.2", "url": "https://raw.githubusercontent.com/hadjiismail123456789-pixel/Tayseer/main/tayseer-updates/bundles/<file>.zip", "checksum": "<sha256>" }
   ```
6. Done. Phones pick it up on next launch and apply it on the following start.
   Bad bundles roll back automatically (the app calls `notifyAppReady()` on boot).

## How the phone decides

`src/lib/github-updates.ts` fetches `latest.json` (raw.githubusercontent, 15s timeout),
compares with the running bundle version (`CapacitorUpdater.current()`), and only when
the published version is strictly newer it downloads (`download({url, version, checksum})`)
and queues it (`next({id})`). Silent when offline or up to date. Native-shell changes
(new plugins/permissions) still require a fresh APK install.
