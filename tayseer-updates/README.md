# Tayseer phone updates — free via GitHub (no Capgo Cloud)

The phone app checks this repo on every launch and stages newer bundles itself.
No account, no subscription. Two files per release:

```
tayseer-updates/
  README.md        <- you are here
  latest.json      <- what phones read: version + zip URL + checksum
```

## Publishing a new update (maintainer)

Every deploy carrying changes bumps the version (package.json, e.g. `1.0.8` ->
`1.0.9`). Phones update when the published version is newer than the running one.

1. Bump `tayseer/package.json` version
2. `cd tayseer` then `npm run build`
3. `npx @capgo/cli bundle zip` -> prints the zip path + sha256 `checksum`
4. Upload the zip to `tayseer-updates/bundles/` in this repo
5. Update `tayseer-updates/latest.json` (raise `version`, new `url` + `checksum`):
   ```json
   { "version": "1.0.9", "url": "https://raw.githubusercontent.com/hadjiismail123456789-pixel/Tayseer/main/tayseer-updates/bundles/<file>.zip", "checksum": "<sha256>" }
   ```
6. Delete the local zip (they accumulate otherwise).
7. Done. Phones stage the new bundle on next launch and apply it on the
   following start. Bad bundles roll back automatically.

## How the phone decides

`src/lib/github-updates.ts` fetches `latest.json` (raw.githubusercontent, 15s timeout),
compares with the running bundle version (`CapacitorUpdater.current()`), and only when
the published version is strictly newer it downloads (`download({url, version, checksum})`)
and queues it (`next({id})`). Silent when offline or up to date. Native-shell changes
(new plugins/permissions) still require a fresh APK install.
