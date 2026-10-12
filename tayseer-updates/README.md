# Tayseer phone updates — free via GitHub (no Capgo Cloud)

The phone app checks this repo on every launch and stages newer bundles itself.
No account, no subscription. Two files per release:

```
tayseer-updates/
  README.md        <- you are here
  latest.json      <- what phones read: version + zip URL + checksum
```

## Publishing a new update (maintainer)

The visible app version (package.json, e.g. `1.0.8`) moves ONLY on real milestones.
Everyday deploys bump the internal OTA build counter instead (`src/lib/build.ts`).

1. Set the next build number in `tayseer/src/lib/build.ts` (`OTA_BUILD`), e.g. `109`
2. `cd tayseer` then `npm run build`
3. `npx @capgo/cli bundle zip` -> prints the zip path + sha256 `checksum`
4. Rename the zip to include the build, e.g. `com.tayseer.quran_1.0.8_b109.zip`
5. Upload the zip to `tayseer-updates/bundles/` in this repo
6. Update `tayseer-updates/latest.json` (keep `version` frozen, raise `build`):
   ```json
   { "version": "1.0.8", "build": 109, "url": "https://raw.githubusercontent.com/hadjiismail123456789-pixel/Tayseer/main/tayseer-updates/bundles/<file>.zip", "checksum": "<sha256>" }
   ```
7. Done. Phones compare `build` numbers and stage newer bundles automatically.
   Bad bundles roll back automatically (the app calls `notifyAppReady()` on boot).

## How the phone decides

`src/lib/github-updates.ts` fetches `latest.json` (raw.githubusercontent, 15s timeout),
compares with the running bundle version (`CapacitorUpdater.current()`), and only when
the published version is strictly newer it downloads (`download({url, version, checksum})`)
and queues it (`next({id})`). Silent when offline or up to date. Native-shell changes
(new plugins/permissions) still require a fresh APK install.
