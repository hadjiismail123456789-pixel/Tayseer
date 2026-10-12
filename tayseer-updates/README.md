# Tayseer phone updates — free via GitHub (no Capgo Cloud)

The phone app checks this repo on every launch and stages newer bundles itself.
No account, no subscription. Two files per release:

```
tayseer-updates/
  README.md        <- you are here
  latest.json      <- what phones read: version + zip URL + checksum
```

## Publishing a new update (maintainer)

No version bumps, no build numbers. The phone updates whenever the published
bundle checksum differs from the running one. The visible app version
(`package.json`) moves ONLY on real milestones.

1. `cd tayseer` then `npm run build`
2. `npx @capgo/cli bundle zip` -> prints the zip path + sha256 `checksum`
3. Upload the zip OVER `tayseer-updates/bundles/com.tayseer.quran_1.0.8.zip`
   in this repo (same filename, Contents API with the file's `sha`)
4. Update `tayseer-updates/latest.json` (same `version`, new `checksum`):
   ```json
   { "version": "1.0.8", "url": "https://raw.githubusercontent.com/hadjiismail123456789-pixel/Tayseer/main/tayseer-updates/bundles/com.tayseer.quran_1.0.8.zip", "checksum": "<new sha256>" }
   ```
5. Delete the local zip (they accumulate otherwise).
6. Done. Phones stage the new bundle on next launch and apply it on the
   following start. Bad bundles roll back automatically.

## How the phone decides

`src/lib/github-updates.ts` fetches `latest.json` (raw.githubusercontent, 15s timeout),
compares with the running bundle version (`CapacitorUpdater.current()`), and only when
the published version is strictly newer it downloads (`download({url, version, checksum})`)
and queues it (`next({id})`). Silent when offline or up to date. Native-shell changes
(new plugins/permissions) still require a fresh APK install.
