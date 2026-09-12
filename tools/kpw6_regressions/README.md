# KPW6 UI and resume regressions

These tests cover UI network preflight, resolver recovery, report timing across
suspend, Home preference persistence, and popup metadata reuse. Fixtures do not
contain accounts, reading history, or device settings. Like actions are mocked.

Copy the repository to `/tmp/miuread-reviewed` on a KOReader device, then run from
the KOReader directory (where `setupkoenv.lua` and `luajit` are available):

```sh
./luajit /tmp/miuread-reviewed/tools/kpw6_regressions/network_test.lua /tmp/miuread-reviewed/miuread.koplugin
./luajit /tmp/miuread-reviewed/tools/kpw6_regressions/service_test.lua /tmp/miuread-reviewed/miuread.koplugin
./luajit /tmp/miuread-reviewed/tools/kpw6_regressions/fork_test.lua /tmp/miuread-reviewed/miuread.koplugin
./luajit /tmp/miuread-reviewed/tools/kpw6_regressions/navigation_test.lua /tmp/miuread-reviewed/miuread.koplugin/main.lua
./luajit /tmp/miuread-reviewed/tools/kpw6_regressions/popup_context_test.lua /tmp/miuread-reviewed/miuread.koplugin/main.lua
```

The first and third tests target Kindle/glibc. The fork test performs DNS lookups
for `weread.qq.com`, but does not call authenticated APIs. Run one instance at a
time. The navigation and popup tests also run under standalone Lua 5.1/LuaJIT.

Verified on KPW6 with beta22:

- Five regression scripts pass, including navigation after a pending real
  setting, stale timers, persistence failure/retry, and empty document paths.
- `python tools/verify_beta22.py`: 292 checks pass.
- Existing `test_online_comment_likes.lua`, `test_extension_download.lua`,
  `test_extension_install.lua`, `test_store_shared.lua`, and
  `test_readtime_recovery.lua` pass. On KOReader, the installer test needs
  `package.loaded.lfs = require('libs/libkoreader-lfs')` after `setupkoenv`.
- Original popup metadata preparation took 426–435 ms; matched-document reuse
  took 0.085–0.170 ms. Subsequent user popup logs recorded 20–49 ms.
- Original navigation-triggered full settings writes took 2.18–2.32 seconds;
  five user section-switch samples after deferring navigation writes took
  355–991 ms, measured through section application with a monotonic clock.

These timings do not include the final physical e-ink refresh. Simulated suspend
and real fork tests do not prove stability after a long real deep-sleep cycle.
Navigation positions remain in memory until a lifecycle save or another settings
write; an abnormal process exit can lose the latest tab/page selection. Reading
progress persistence is unchanged.
