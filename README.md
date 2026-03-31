# CalDietAI — Food Assets CDN

This repository hosts the CalDietAI full food database file, served via **jsDelivr CDN**.

- **Branch:** `assets` — contains only the binary data file
- **CDN:** `https://cdn.jsdelivr.net/gh/nhanntv/ntv-caldietai-assets-foods@v<N>/cal_diet_ai_foods_full.gz`
- **Version control:** Firebase Remote Config (`food_db.full_version` + `food_db.full_url`)

> Tag name matches `full_version` in Remote Config — e.g. tag `v1` = `full_version: 1`.

---

## File

| File | Description | Size |
|------|-------------|------|
| `cal_diet_ai_foods_full.gz` | Full food database (~325K foods, 9 regional sources) | ~15 MB |

---

## Releasing a New Version

### Prerequisites

- Access to this GitHub repo
- New `cal_diet_ai_foods_full.gz` generated from `FoodDataset/consolidate.py`
- Access to Firebase Console → Remote Config

### Steps

**1. Switch to the `assets` branch**

```bash
git checkout assets
```

**2. Replace the file**

```bash
cp /path/to/new/cal_diet_ai_foods_full.gz .
```

> On the CalDietAI dev machine, the source file is at:
> `/Users/tdnhan/Projects/Individuals/CalDietAi/CalDietAi-mobile/assets/data/cal_diet_ai_foods_full.gz`

**3. Commit and tag**

Use the next version number — tag name must match `full_version` in Remote Config (e.g. releasing v2):

```bash
git add cal_diet_ai_foods_full.gz
git commit -m "food db v2"
git tag v2
git push origin assets
git push origin v2
```

**4. Verify the CDN URL**

Open in browser — should start downloading (allow a few minutes for jsDelivr to cache):

```
https://cdn.jsdelivr.net/gh/nhanntv/ntv-caldietai-assets-foods@v2/cal_diet_ai_foods_full.gz
```

**5. Update Firebase Remote Config**

Go to Firebase Console → Remote Config → edit key `app_remote_config`:

```json
{
  "food_db": {
    "full_version": 2,
    "full_url": "https://cdn.jsdelivr.net/gh/nhanntv/ntv-caldietai-assets-foods@v2/cal_diet_ai_foods_full.gz"
  }
}
```

Publish the changes. The app will detect the new version on next launch and download in background.

---

## Version History

| Tag | `full_version` | Date | Notes |
|-----|---------------|------|-------|
| `v1` | 1 | 2026-03-31 | Initial release |

> Update this table each time you release a new version.

---

## How the App Uses This File

1. App fetches `food_db.full_version` and `food_db.full_url` from Firebase Remote Config on launch.
2. Compares remote version against locally stored version (`SharedPreferences`).
3. If remote > local → downloads `.gz` from CDN URL in background (home screen).
4. Decompresses with `gzip.decode()` → imports ~325K foods into local SQLite DB.
5. Falls back to Firebase Storage if CDN download fails.

See `lib/core/database/food_sync_service.dart` in the mobile repo for implementation details.
