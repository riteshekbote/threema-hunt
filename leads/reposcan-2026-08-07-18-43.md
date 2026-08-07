
===== REPOSCAN 2026-08-07 18:43:08 UTC =====
## Grep hits:
SCAN SUMMARY: 6656 code/config files scanned, 56 hits
reposcan-raw/threema-ch/threema-ios/Threema/AppMigration.swift:873: let persistenceKeyLicensePassword = "Threema license password"
reposcan-raw/threema-ch/threema-ios/ThreemaFrameworkTests/SafeDataTests.swift:11: serverPassword: "password1",
reposcan-raw/threema-ch/threema-ios/ThreemaFrameworkTests/HTTPClientTests.swift:15: let expectedAuthToken = "authToken"
reposcan-raw/threema-ch/threema-ios/ThreemaFrameworkTests/HTTPClientTests.swift:18: let expectedPassword = "testPassword"
reposcan-raw/threema-ch/threema-ios/ThreemaFrameworkTests/WorkDataFetchTests/WorkDataThreemaMDMFetcherTests.swift:50: licenseStoreMock.licensePassword = "testPassword"
reposcan-raw/threema-ch/threema-ios/ThreemaFrameworkTests/WorkDataFetchTests/WorkDataThreemaMDMFetcherTests.swift:72: (username: nil as String?, password: "testPassword" as String?, description: "nil username"),
reposcan-raw/threema-ch/threema-ios/ThreemaFrameworkTests/WorkDataFetchTests/WorkDataThreemaMDMFetcherTests.swift:148: licenseStoreMock.licensePassword = "testPassword"
reposcan-raw/threema-ch/threema-ios/ThreemaFrameworkTests/WorkDataFetchTests/WorkDataAPICallerTests.swift:13: let expectedPassword = "testPassword"
reposcan-raw/threema-ch/threema-ios/ThreemaFrameworkTests/WorkDataFetchTests/WorkDataAPICallerTests.swift:37: (username: nil as String?, password: "validPassword" as String?, description: "nil username"),
reposcan-raw/threema-ch/threema-ios/ThreemaFrameworkTests/WorkDataFetchTests/WorkDataAPICallerTests.swift:103: password: "testPassword",
reposcan-raw/threema-ch/threema-ios/ThreemaTests/SafeTests.swift:67: let result = SafeStore.createKey(identity: "ECHOECHO", safePassword: "shootdeathstar")
reposcan-raw/threema-ch/threema-ios/ThreemaTests/SafeTests.swift:76: let key = SafeStore.createKey(identity: "ECHOECHO", safePassword: "shootdeathstar")
reposcan-raw/threema-ch/threema-ios/ThreemaTests/SafeTests.swift:105: let key = SafeStore.createKey(identity: "ECHOECHO", safePassword: "shootdeathstar")
reposcan-raw/threema-ch/threema-ios/ThreemaTests/MyProfile/ThreemaSafe/ThreemaSafePasswordValidatorTests.swift:30: let password = "abcdefghijkl"
reposcan-raw/threema-ch/threema-ios/ThreemaTests/MyProfile/ThreemaSafe/ThreemaSafePasswordValidatorTests.swift:227: let validPassword = "ABcd!@#tgk092"
reposcan-raw/threema-ch/threema-ios/ThreemaTests/ManuallyTests/ThreemaSafe/HttpClientDownloadSafeTests.swift:28: if let key = SafeStore.createKey(identity: "ECHOECHO", safePassword: "shootdeathstar"),
reposcan-raw/threema-ch/threema-ios/ThreemaTests/ManuallyTests/ThreemaSafe/HttpClientDownloadSafeTests.swift:76: if let key = SafeStore.createKey(identity: "ECHOECHO", safePassword: "shootdeathstar"),
reposcan-raw/threema-ch/threema-ios/ThreemaTests/ManuallyTests/ThreemaSafe/HttpClientDownloadSafeTests.swift:120: safePassword: "shootdeathstar"
reposcan-raw/threema-ch/threema-ios/ThreemaTests/ManuallyTests/ThreemaSafe/HttpClientUploadSafeTests.swift:38: if let key = SafeStore.createKey(identity: "ECHOECHO", safePassword: "shootdeathstar"),
reposcan-raw/threema-ch/threema-ios/ThreemaTests/ManuallyTests/ThreemaSafe/HttpClientUploadSafeTests.swift:83: if let key = SafeStore.createKey(identity: "ECHOECHO", safePassword: "shootdeathstar"),
reposcan-raw/threema-ch/threema-ios/libthreema/threema-protocols/src/directory.openapi.yml:7: - url: https://ds-apip-work.test.threema.ch
reposcan-raw/threema-ch/threema-ios/libthreema/threema-protocols/src/directory.openapi.yml:917: - url: https://ds-apip.test.threema.ch
reposcan-raw/threema-ch/threema-android/domain/src/main/java/ch/threema/domain/protocol/api/SfuToken.kt:49: """sfuToken='********', expirationDate=$expirationDate)"""
reposcan-raw/threema-ch/threema-android/domain/protocol/src/directory.openapi.yml:7: - url: https://ds-apip-work.test.threema.ch
reposcan-raw/threema-ch/threema-android/domain/protocol/src/directory.openapi.yml:911: - url: https://ds-apip.test.threema.ch
reposcan-raw/threema-ch/threema-android/domain/libthreema/threema-protocols/src/directory.openapi.yml:7: - url: https://ds-apip-work.test.threema.ch
reposcan-raw/threema-ch/threema-android/domain/libthreema/threema-protocols/src/directory.openapi.yml:917: - url: https://ds-apip.test.threema.ch
reposcan-raw/threema-ch/threema-android/app/src/main/java/ch/threema/storage/models/WebClientSessionModel.java:28: public static final String COLUMN_PUSH_TOKEN = "push_token";
reposcan-raw/threema-ch/threema-android/app/src/main/java/ch/threema/storage/databaseupdate/DatabaseUpdateToVersion37.java:48: static final String COLUMN_PUSH_TOKEN = "push_token";
reposcan-raw/threema-ch/threema-android/app/src/main/java/ch/threema/app/utils/IceCandidateParser.java:43: public final static String TOKEN = "[a-zA-Z]+";
reposcan-raw/threema-ch/threema-android/app/src/main/java/ch/threema/app/webclient/converter/ClientInfo.java:37: private final static String PUSH_TOKEN = "pushToken";
reposcan-raw/threema-ch/threema-android/app/src/main/java/ch/threema/app/voip/groupcall/sfu/JoinResponse.kt:70: "iceUsernameFragment='$iceUsernameFragment', icePassword='$icePassword', " +
reposcan-raw/threema-ch/threema-android/app/src/main/java/ch/threema/app/notifications/NotificationChannels.kt:52: const val NOTIFICATION_CHANNEL_REMOTE_SECRET = "remote_secret_monitor"
reposcan-raw/threema-ch/threema-android/app/src/main/java/ch/threema/app/dialogs/PasswordEntryDialog.java:47: public static final @NonNull String KEY_PASSWORD = "password";
reposcan-raw/threema-ch/threema-android/buildSrc/src/main/kotlin/config/SentryConfig.kt:15: publicApiKey = "b3e20afbf356a8748bb62ac165aa780c",
reposcan-raw/threema-ch/threema-android/buildSrc/src/main/kotlin/config/SentryConfig.kt:19: publicApiKey = "615af77cb3d980c41b3b04b07417cc7d",
reposcan-raw/threema-ch/threema-desktop/apps/desktop/config/vite.config.ts:147: DIRECTORY_SERVER_URL: 'https://ds-apip.test.threema.ch/',
reposcan-raw/threema-ch/threema-desktop/apps/desktop/config/vite.config.ts:153: WORK_SERVER_LEGACY_URL: 'https://ds-apip-work.test.threema.ch/',
reposcan-raw/threema-ch/threema-desktop/apps/desktop/config/vite.config.ts:154: WORK_SERVER_URL: 'https://work.test.threema.ch/',

## Model analysis:
[0m
> build · nemotron-3-ultra-free
[0m
[0m→ [0mRead reposcan-raw/threema-ch/threema-desktop/apps/desktop/src/common/node/key-storage/crypto.ts
[0m→ [0mRead reposcan-raw/threema-ch/threema-desktop/apps/desktop/src/common/test-data.ts[90m [offset=55, limit=20][0m
Now I have full context. Let me classify all hits:

---

### **REAL_SECRET** 
| File | Line | SHA256 (secret) | Verdict |
|------|------|-----------------|---------|
| *(none)* | | | No hardcoded production credentials found |

---

### **TEST_OR_EXAMPLE**
| File | Line | Context |
|------|------|---------|
| threema-android/.../PasswordEntryDialog.java | 47 | `KEY_PASSWORD = "password"` — constant key name |
| threema-android/.../NotificationChannels.kt | 52 | `NOTIFICATION_CHANNEL_REMOTE_SECRET = "remote_secret_monitor"` — channel name |
| threema-android/.../IceCandidateParser.java | 43 | `TOKEN = "[a-zA-Z]+"` — regex pattern |
| threema-android/.../JoinResponse.kt | 70 | Variable refs in string template (`icePassword`) |
| threema-android/.../ClientInfo.java | 37 | `PUSH_TOKEN = "pushToken"` — constant key name |
| threema-android/.../DatabaseUpdateToVersion37.java | 48 | `COLUMN_PUSH_TOKEN = "push_token"` — DB column |
| threema-android/.../WebClientSessionModel.java | 28 | `COLUMN_PUSH_TOKEN = "push_token"` — DB column |
| threema-android/.../SfuToken.kt | 49 | `sfuToken='********'` — masked in toString |
| threema-desktop/.../enum/index.ts | 3126,3162 | `LOAD_USER_PASSWORD` / `STORE_USER_PASSWORD` — enum values |
| threema-desktop/.../schema.ts | 983,1000 | Same enum values (source of generated file) |
| threema-desktop/.../test-data.ts | 62 | `keyStoragePassword: 'CHANGE_ME'` — explicit placeholder |
| threema-desktop/.../ice-password.ts | 12 | Error message template |
| threema-ios/.../AppMigration.swift | 873 | `persistenceKeyLicensePassword = "Threema license password"` — storage key name |
| **All test files** (HTTPClientTests, SafeDataTests, WorkDataAPICallerTests, WorkDataThreemaMDMFetcherTests, HttpClientDownloadSafeTests, HttpClientUploadSafeTests, SafeTests, ThreemaSafePasswordValidatorTests) | Various | Test fixtures: `"testPassword"`, `"authToken"`, `"password1"`, `"shootdeathstar"`, `"abcdefghijkl"`, `"ABcd!@#tgk092"`, `"validPassword"` |

---

### **ENDPOINT_LEAK**
| File | Line | Endpoint |
|------|------|----------|
| threema-android/domain/libthreema/threema-protocols/src/directory.openapi.yml | 7, 917 | `https://ds-apip-work.test.threema.ch`, `https://ds-apip.test.threema.ch` |
| threema-android/domain/protocol/src/directory.openapi.yml | 7, 911 | Same test/staging directory service URLs |
| threema-ios/libthreema/threema-protocols/src/directory.openapi.yml | 7, 917 | Same test/staging directory service URLs |
| threema-desktop/packages/libthreema-wasm/libs/libthreema/threema-protocols/src/directory.openapi.yml | 7, 911 | Same test/staging directory service URLs |
| threema-desktop/apps/desktop/config/vite.config.ts | 147, 153, 154 | `DIRECTORY_SERVER_URL`, `WORK_SERVER_LEGACY_URL`, `WORK_SERVER_URL` pointing to test env |

---

### **INTERESTING**
| File | Line | Finding |
|------|------|---------|
| threema-android/buildSrc/src/main/kotlin/config/SentryConfig.kt | 15 | Sentry public DSN key: `b3e20afbf356a8748bb62ac165aa780c` (public by design) |
| threema-android/buildSrc/src/main/kotlin/config/SentryConfig.kt | 19 | Sentry public DSN key: `615af77cb3d980c41b3b04b07417cc7d` (public by design) |
| threema-desktop/apps/desktop/src/common/node/key-storage/crypto.ts | 223 | Benchmark constant `'r3gGN9GDQ5NF6tM6'` — **fixed password used only in `determineKdfParams()` to calibrate Argon2id parameters for user's hardware at runtime; not used for actual encryption** |

---

### **VERDICT LIST**

| Candidate | REPORT_CANDIDATE | Reason |
|-----------|------------------|--------|
| Sentry DSN keys (Android) | **no** | Public keys by design (DSN public portion) |
| Benchmark password `r3gGN9GDQ5NF6tM6` (Desktop) | **no** | Hardcoded constant for KDF calibration benchmark only; actual user password supplied at runtime |
| Test/staging directory URLs (`ds-apip.test.threema.ch`, `ds-apip-work.test.threema.ch`, `work.test.threema.ch`) | **no** | Known test endpoints documented in OpenAPI specs and Vite config; not production secrets |
| All test fixtures (`testPassword`, `shootdeathstar`, `CHANGE_ME`, etc.) | **no** | Explicit test/fixture data |
| All constant key names (`KEY_PASSWORD`, `PUSH_TOKEN`, `COLUMN_PUSH_TOKEN`, enum values) | **no** | Identifier constants, not secret values |

**Summary: No reportable secrets found in this delta.** The only "live-looking" string (`r3gGN9GDQ5NF6tM6`) is a benchmark fixture for Argon2id parameter tuning, not a production credential.
