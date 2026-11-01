# Core KenshiCoop changes and integration map

Snapshot of public branches on **November 1, 2026**. Covers nhoral, Savas07, all five ximery4 branches, both cookingwithsimon and Wo-o-o branches, captainjoshua77-wq and DerpyNewb. Source/DLLs/test scripts were not executed. Checks covered the git graph, complete diffs, function hunks, Wo↔Savas patch IDs and mechanical application of the embedded Captain patch through a temporary index. Unless stated otherwise, potential defects are static findings, not reproduced gameplay failures.

Recommended base: [Savas07 `58514ea`](https://github.com/Savas07/KenshiCoop/commit/58514ea33538fca78e932abf44bb658d4e915236), retaining its history over [nhoral `5a761e19`](https://github.com/nhoral/KenshiCoop/commit/5a761e19a4184b42315e96ea7e92e3c1403d54db). This is **73 commits including nine merges**, 91 changed files, 22,640 insertions and 1,951 deletions. It expands v0.51 with eight-player networking, routing, lobbies, authority and inventory changes. Preserve the final base instead of reconstructing it through isolated older fixes.

[changes-core.json](changes-core.json) records 31 feature IDs, 93 distinct commits in this subset and 16 branch tips, with parents, author/committer dates, files, function contexts and feature assignments. All 73 Savas commits are covered. Uniqueness assertions here apply only to this inspected subset; the consolidated map handles other families.

## Git facts and duplicate prevention

- ximery4/main is exactly Savas07/main at `58514ea`; no second import is needed.
- DerpyNewb/main, cookingwithsimon/main and Wo-o-o/main exactly match upstream `5a761e19`.
- All three other nhoral branches (`feature/wanderer-x2-start`, `fix/inventory-loss-stabilization`, `spikes`) are ancestors of main; main..branch contains zero commits.
- Both cookingwithsimon branch commits `768d25b`/`0c493a9` were inspected: only `.gitignore` (`local/`) and NOTES.md, with no runtime changes beneath the documentation tip.
- Wo-o-o/dev/stabilize-v0.51 has six commits: four runtime fixes, then CI and documentation. No full patch ID among these six equals an individual Savas commit; partial semantic overlap still exists.
- ximery4/xfer-census-source-fab (`d253e78` + `6e62ceb`) and upstream/xfer-fixture-fab (`5f89ce8`) have identical complete trees. Count once.
- Old ximery4/trader-identity (`e44ec65`) has separate root `83336f8`, no common ancestor with Savas: imported v0.58.1 plus patches, removed dist/mod-kit and added third_party/KenshiLib_v0.5.0 gitlink. Use clean upstream/* branches instead of merging that root.
- Captain main stores ci/fix.patch only. Its internal patch SHA `0f187774...` differs from repository commit `fc90aff...`; storing the patch does not apply it to the build.

## Functional groups

All SAV-* groups are retained through the selected base, not cherry-picked again. Listed SHAs anchor each group; the complete covered history appears below and in JSON. Preserve final behavior, scenarios and follow-up corrections, rather than simultaneously enabling old WIP/reversed designs.

### SAV-NET8 — 8-player star topology, relay, identity/player masks, per-player squad ranks

- Planned action: `included_base`.
- Source commits: [a5b12518](https://github.com/Savas07/KenshiCoop/commit/a5b1251891bb7022a33cc09652aad88daacc7863).
- Files: `src/netproto/Wire.h`, `src/plugin/net/NetLink.cpp`, `src/plugin/net/NetLink.h`, `src/plugin/net/SteamP2P.cpp`, `src/plugin/core/OwnRanks.h`, `src/plugin/core/Inbound.h`, `src/plugin/Plugin.cpp`.
- Symbols: `MAX_PLAYERS`, `PEER_MASK_ALL`, `NetLink::allocPlayerId`, `NetLink::threadLoop`, `resolveOwnRanksForPlayer`.
- Dependencies: —.

### SAV-LOBBY — Steam invites, main-menu lobby, ready-world admission, join without local saves

- Planned action: `included_base`.
- Source commits: [a5b12518](https://github.com/Savas07/KenshiCoop/commit/a5b1251891bb7022a33cc09652aad88daacc7863), [6df1084e](https://github.com/Savas07/KenshiCoop/commit/6df1084e8f99bdc28b3379f84b0cb43d759681f3), [d7a3cc07](https://github.com/Savas07/KenshiCoop/commit/d7a3cc07c6362a79e73a999d4d13c6965bd53b51), [de1d5c22](https://github.com/Savas07/KenshiCoop/commit/de1d5c22dbf247d52d79410a2145595191c006e4), [aaea80a6](https://github.com/Savas07/KenshiCoop/commit/aaea80a6ec3f12bdb408d5ba1c6b7b05015e3474).
- Files: `src/plugin/net/SteamInvite.cpp`, `src/plugin/game/EngineLobby.cpp`, `src/plugin/game/EngineUi.cpp`, `src/plugin/Plugin.cpp`.
- Symbols: `onLobbyEnter`, `onLobbyCreated`, `coopPanelDrive`, `titleUpdate_hook`, `noteWorldLive`.
- Dependencies: `SAV-NET8`.

### SAV-CHARGEN — Native join character editor, fresh-start gate, transmitted appearance

- Planned action: `included_base`.
- Source commits: [2e1f5ce4](https://github.com/Savas07/KenshiCoop/commit/2e1f5ce421272a899950c72936a19715110394c8), [433f3275](https://github.com/Savas07/KenshiCoop/commit/433f3275ed7c46a2a4175cd129b4182ffdb51d94), [6fb51c09](https://github.com/Savas07/KenshiCoop/commit/6fb51c092b584bc7cbd34718fe50cfac9f162e35), [de1d5c22](https://github.com/Savas07/KenshiCoop/commit/de1d5c22dbf247d52d79410a2145595191c006e4), [de302373](https://github.com/Savas07/KenshiCoop/commit/de3023734003a35c62d3a74db750481e128a70c7).
- Files: `src/netproto/Wire.h`, `src/plugin/game/EngineCharGen.cpp`, `src/plugin/game/EngineInternal.cpp`, `src/plugin/Plugin.cpp`, `src/plugin/net/NetLink.cpp`.
- Symbols: `CharGenHeader`, `tickCharGen`, `chargenAutoConfirm`, `LoadGoPacket::freshStart`, `pumpSaveReceive`.
- Dependencies: `SAV-NET8`, `SAV-SAVE`.

### SAV-ROSTER — Steam identity persisted in save, named squad tabs, absent-player substitution

- Planned action: `included_base`.
- Source commits: [be1a0519](https://github.com/Savas07/KenshiCoop/commit/be1a05193b8578b683a8dc704f341dafd31e6618), [3a567b92](https://github.com/Savas07/KenshiCoop/commit/3a567b9259017bddbe7c11a4fd6f65055e7caa6a), [29090d90](https://github.com/Savas07/KenshiCoop/commit/29090d908778ad775068dd3d210c8cd3a908af10).
- Files: `src/plugin/net/NetLink.cpp`, `src/plugin/game/EngineEntity.cpp`, `src/plugin/Plugin.cpp`, `src/netproto/Wire.h`.
- Symbols: `RosterPacket`, `NetLink::allocPlayerId`, `applyHostRanks`, `publishOwned`.
- Dependencies: `SAV-NET8`.

### SAV-SAVE — Unicode save paths, bootstrap name, receive ordering, safe atomic commit and recovery

- Planned action: `included_base`.
- Source commits: [88ec9518](https://github.com/Savas07/KenshiCoop/commit/88ec95183e74c9d6977d660def6e87f318cca48d), [6fb51c09](https://github.com/Savas07/KenshiCoop/commit/6fb51c092b584bc7cbd34718fe50cfac9f162e35), [de1d5c22](https://github.com/Savas07/KenshiCoop/commit/de1d5c22dbf247d52d79410a2145595191c006e4), [0c90ef83](https://github.com/Savas07/KenshiCoop/commit/0c90ef83f1c7dcd1b38f04ac28d664afe18bb07e).
- Files: `src/plugin/sync/SaveXfer.cpp`, `src/plugin/sync/SaveXfer.h`, `src/plugin/Plugin.cpp`, `src/prototest/main.cpp`.
- Symbols: `walkFolder`, `tickSend`, `onSaveBegin`, `onSaveFile`, `onSaveDone`, `recoverStrandedSave`, `pumpSaveReceive`.
- Dependencies: —.

### SAV-DRIVE — Movement sync, glide/get-up recovery, pose/carry guards, Steam transport guards

- Planned action: `included_base`.
- Source commits: [433f3275](https://github.com/Savas07/KenshiCoop/commit/433f3275ed7c46a2a4175cd129b4182ffdb51d94), [ec897642](https://github.com/Savas07/KenshiCoop/commit/ec89764294f50e1ded464c015acb828bd2ce783c), [86b95e08](https://github.com/Savas07/KenshiCoop/commit/86b95e08f866277ec3e4e0740956b6dee384b9f8), [4adca37c](https://github.com/Savas07/KenshiCoop/commit/4adca37c471e1a3e0382393b52818d596b773567), [fa71c222](https://github.com/Savas07/KenshiCoop/commit/fa71c2220ace43c7b6c5ba8021ad76caa6c17be5).
- Files: `src/plugin/sync/ReplicatorDrive.cpp`, `src/plugin/sync/Interp.h`, `src/plugin/sync/ReplicatorUtil.h`, `src/plugin/game/EngineCharState.cpp`, `src/plugin/net/SteamP2P.cpp`.
- Symbols: `Replicator::applyTargets`, `Replicator::pruneDriveGrace`, `taskIsGetUp`.
- Dependencies: `SAV-NET8`.

### SAV-DEVTOOLS — Opt-in overlay, recording/replay, watchdog, telemetry dashboard and repro commands

- Planned action: `included_base_opt_in`.
- Source commits: [4f185bcc](https://github.com/Savas07/KenshiCoop/commit/4f185bcc20e877b06988127ff042658f551422d3), [fff395ae](https://github.com/Savas07/KenshiCoop/commit/fff395aeaa49b740c10fc6df6d4e8491ad5f03a3).
- Files: `src/plugin/devtools/DevTools.cpp`, `src/plugin/devtools/DevOverlay.cpp`, `src/plugin/devtools/PacketRecorder.cpp`, `src/plugin/sync/ReplicatorDev.cpp`, `src/plugin/core/Config.h`, `tools/devkit/dashboard/server.py`, `tools/devkit/replay/kcreplay.cpp`.
- Symbols: `KENSHICOOP_DEVTOOLS`, `KENSHICOOP_DEV_OVERLAY`, `KENSHICOOP_DEV_RECORD`, `KENSHICOOP_DEV_WATCHDOG`.
- Dependencies: —.

### SAV-VENDOR — Host-authoritative shop counters/cash, buy placement once, vendor oracles

- Planned action: `included_base`.
- Source commits: [3198745f](https://github.com/Savas07/KenshiCoop/commit/3198745f76e67edf6a64f9dbe68eefcd6e636075), [19e2f625](https://github.com/Savas07/KenshiCoop/commit/19e2f62546cea77308515c6d557f9d4860e5f7f6), [ebbc6f0e](https://github.com/Savas07/KenshiCoop/commit/ebbc6f0eaee3d818d9e6c4292c4fa7b8ed3a6337), [e6e1ddde](https://github.com/Savas07/KenshiCoop/commit/e6e1dddebfd2098722a21aaff4d2cffb4bab1f39), [7c6e19c3](https://github.com/Savas07/KenshiCoop/commit/7c6e19c387035cfb8620b797601f86fdd67819a3).
- Files: `src/netproto/Wire.h`, `src/plugin/game/EngineInventory.cpp`, `src/plugin/game/EngineWorld.cpp`, `src/plugin/sync/ReplicatorItems.cpp`, `src/plugin/net/NetLink.cpp`.
- Symbols: `publishInventories`, `detectAndPublishVendorDeltas`, `applyVendorDeltas`, `applyVendorCash`, `vendorBuyInto`, `placeOnce`.
- Dependencies: `SAV-NET8`.

### SAV-PRODUCTION — Single host production/crafting simulation and machine pose healing

- Planned action: `included_base`.
- Source commits: [e5ad2ef0](https://github.com/Savas07/KenshiCoop/commit/e5ad2ef0a85fdecd88bbd9bb8b96d9c58fcec4ba), [f80779ed](https://github.com/Savas07/KenshiCoop/commit/f80779edf95b9725890240114c211223422c2f66).
- Files: `src/plugin/game/EngineInternal.cpp`, `src/plugin/game/EngineWorld.cpp`, `src/plugin/sync/ReplicatorChannels.cpp`, `src/plugin/sync/ReplicatorDrive.cpp`.
- Symbols: `operateProd_hook`, `operateCraft_hook`, `opHostSimulates`, `installMachineOperateHooks`, `applyProd`, `applyRest`.
- Dependencies: `SAV-NET8`.

### SAV-RECRUIT — Recruit identity translation, owner pinning, no phantom NPC duplicate or stale AI walk

- Planned action: `included_base`.
- Source commits: [0b8f63f8](https://github.com/Savas07/KenshiCoop/commit/0b8f63f8de94288ed12cf5e13738ab6007b1c416), [3e9bd90b](https://github.com/Savas07/KenshiCoop/commit/3e9bd90b66188bd99ac08e8d873389f2b9bab86a), [5c5f3886](https://github.com/Savas07/KenshiCoop/commit/5c5f3886f93769a3a795ac48a81eb73fb6723328).
- Files: `src/plugin/game/EngineInternal.cpp`, `src/plugin/sync/ReplicatorChannels.cpp`, `src/plugin/sync/ReplicatorPublish.cpp`, `src/plugin/sync/ReplicatorDrive.cpp`.
- Symbols: `recruit_hook`, `publishRecruits`, `releaseRecruitDrive`, `publishOwned`, `applyTargets`.
- Dependencies: `SAV-NET8`.

### SAV-COMBAT — Body-resolved combat subjects, reissue budget, engaged copy snap/walk exclusion

- Planned action: `included_base`.
- Source commits: [337004a2](https://github.com/Savas07/KenshiCoop/commit/337004a2873dfb1e0f8077af596f0799d447e44b), [5d845833](https://github.com/Savas07/KenshiCoop/commit/5d845833c0baedf62aade21fdba85be3c856072c), [65b81d53](https://github.com/Savas07/KenshiCoop/commit/65b81d53996b2e87b1a214952d4d20e0cdf98f7c), [d2509ed3](https://github.com/Savas07/KenshiCoop/commit/d2509ed32ac7dc24f0a394374ba8f1445b115af3), [ef40acd5](https://github.com/Savas07/KenshiCoop/commit/ef40acd5c3b01f6e3c99ef2f8db42c9de160eaf0).
- Files: `src/plugin/game/EngineSpawnCombat.cpp`, `src/plugin/sync/ReplicatorDrive.cpp`, `src/plugin/sync/ReplicatorUtil.h`.
- Symbols: `applyCombat`, `forceAttack`, `applyTargets`, `pruneDriveGrace`.
- Dependencies: `SAV-DRIVE`.
- Integration notes: Historical reversals 65b81d5/d2509ed are not two desired simultaneous policies; preserve final ef40acd behavior.

### SAV-WORLD — Structural native baseline, delayed/dormant drop tracking, native pickup notification

- Planned action: `included_base`.
- Source commits: [d82a496d](https://github.com/Savas07/KenshiCoop/commit/d82a496d8ac1994f008818394728e82244a174aa).
- Files: `src/netproto/Wire.h`, `src/plugin/sync/ReplicatorItems.cpp`, `src/plugin/sync/Replicator.h`, `src/plugin/net/NetLink.cpp`, `src/plugin/game/EngineInventory.cpp`.
- Symbols: `NativeTakenPacket`, `publishWorldItems`, `applyWorldClaims`, `applyNativeTakes`, `rememberDrop`, `takeRecentDrop`.
- Dependencies: `SAV-NET8`.

### SAV-CORPSE — Dead non-squad inventory census keyed by streamed body identity

- Planned action: `included_base`.
- Source commits: [0c3def37](https://github.com/Savas07/KenshiCoop/commit/0c3def37aca7f58ecd40fb3bc799489428f61fcf).
- Files: `src/plugin/game/EngineWorld.cpp`, `src/plugin/sync/ReplicatorItems.cpp`, `src/plugin/core/Config.h`, `src/netproto/Wire.h`.
- Symbols: `INV_FLAG_CORPSE`, `publishInventories`, `ingestInv`, `censusCorpses_`.
- Dependencies: `SAV-VENDOR`.

### SAV-RESEARCH — Grow-only research union, joining player bench costs reach host

- Planned action: `included_base`.
- Source commits: [a6c97be0](https://github.com/Savas07/KenshiCoop/commit/a6c97be05cddc63fda4f8aade029a67814126cbf).
- Files: `src/plugin/sync/ReplicatorChannels.cpp`, `src/plugin/sync/ReplicatorItems.cpp`, `src/plugin/game/EngineInventory.cpp`, `src/netproto/Wire.h`.
- Symbols: `publishResearch`, `applyResearch`, `detectAndPublishVendorDeltas`, `researchPickSubject`.
- Dependencies: `SAV-VENDOR`.

### SAV-JOINSTATE — Persist joining player squad/faction names and permanent jobs in host save

- Planned action: `included_base`.
- Source commits: [76975155](https://github.com/Savas07/KenshiCoop/commit/769751554310e1976e42155d4ac81457f9dd02e7).
- Files: `src/netproto/Wire.h`, `src/plugin/sync/ReplicatorChannels.cpp`, `src/plugin/game/EngineCharState.cpp`, `src/plugin/game/EngineInternal.cpp`, `src/plugin/net/NetLink.cpp`.
- Symbols: `JoinStatePacket`, `JS_SQUAD_NAME`, `JS_FACTION`, `JS_PERMAJOBS`, `forgetPeerChannels`.
- Dependencies: `SAV-NET8`, `SAV-ROSTER`.

### SAV-XFER-RACE — Refuse authoritative source shortfall and converge rejected/partial optimistic loot

- Planned action: `included_base`.
- Source commits: [32440706](https://github.com/Savas07/KenshiCoop/commit/32440706db1c86789dc467608ad7b9a4c61c3f6f).
- Files: `src/plugin/sync/ReplicatorItems.cpp`, `src/plugin/test/ScenarioInventory.cpp`.
- Symbols: `applyTransfers`, `applyXferAcks`, `corpse_loot_race`.
- Dependencies: `SAV-CORPSE`.

### SAV-GEAR — Drop the actual moved gear object and notice last-item drop from bag

- Planned action: `included_base`.
- Source commits: [4bd312f8](https://github.com/Savas07/KenshiCoop/commit/4bd312f8b9d597a57024afebc6955cc750404668).
- Files: `src/plugin/game/EngineInventory.cpp`, `src/plugin/sync/ReplicatorItems.cpp`, `src/plugin/test/ScenarioInventory.cpp`.
- Symbols: `dropItemFromInventory`, `detectAndPublishWeaponDrops`.
- Dependencies: —.

### SAV-TRADER — Host merchant lock and absolute touched shop lines/money handover

- Planned action: `included_base`.
- Source commits: [19497e70](https://github.com/Savas07/KenshiCoop/commit/19497e70c0a27c6642ae6285d79bf61ed4baefd2), [3ec2551a](https://github.com/Savas07/KenshiCoop/commit/3ec2551af293eb0e55bbafe7e97854a976d59106).
- Files: `src/netproto/Wire.h`, `src/plugin/sync/ReplicatorItems.cpp`, `src/plugin/game/EngineWorld.cpp`, `src/plugin/game/EngineInventory.cpp`, `src/plugin/net/NetLink.cpp`.
- Symbols: `TradeHeader`, `tickTradeLock`, `sendTradeOp`, `applyTradeTable`, `applyShop`, `closeTradeWindow`.
- Dependencies: `SAV-VENDOR`.

### SAV-IDENTITY-BAGS — v0.58.1 bag creation grace, stable publish identities, translated medical/carry, authority fixes

- Planned action: `included_base`.
- Source commits: [58514ea3](https://github.com/Savas07/KenshiCoop/commit/58514ea33538fca78e932abf44bb658d4e915236).
- Files: `src/plugin/sync/ReplicatorPublish.cpp`, `src/plugin/sync/ReplicatorAuthority.cpp`, `src/plugin/sync/ReplicatorChannels.cpp`, `src/plugin/sync/ReplicatorItems.cpp`, `src/plugin/sync/ReplicatorSpawn.cpp`, `src/plugin/game/EngineInventory.cpp`.
- Symbols: `indexProxies`, `wireKeyOf`, `publishKeyFor`, `pruneLastHands`, `xlateSubjects`, `identityKeyOf`, `bodyForKey`, `syncCellClaims`, `parkDivergedCopy`, `syncSpeed`, `BAG_CREATE_GRACE_MS`, `applyContainerContents`.
- Dependencies: `SAV-RECRUIT`, `SAV-DRIVE`, `SAV-COMBAT`.

### SAV-SUPPORT — All accompanying scenarios, oracles, debug probes, version/UI/docs/packaging and merge history

- Planned action: `included_base_rebuild_artifacts`.
- Source commits: [1c147364](https://github.com/Savas07/KenshiCoop/commit/1c147364c5886dd0ef8cd99d0d6efe1608e7b270), [c939bcce](https://github.com/Savas07/KenshiCoop/commit/c939bcce42aa0d77b9341c032efe216b8156b3de), [f76021a4](https://github.com/Savas07/KenshiCoop/commit/f76021a42376e30c485202c0c6f15f575421fd88), [79e4645f](https://github.com/Savas07/KenshiCoop/commit/79e4645f1198a2e6275b46484e7d7a1c21a8511a), [73518a01](https://github.com/Savas07/KenshiCoop/commit/73518a017e1b3b1672b3e12d0b5adbfb7218d815), [74049b22](https://github.com/Savas07/KenshiCoop/commit/74049b224724df52777184beeb947f7ea14c958f), [01f9d227](https://github.com/Savas07/KenshiCoop/commit/01f9d22711386f4a8911a09059a5539313040150), [eecb4e2e](https://github.com/Savas07/KenshiCoop/commit/eecb4e2e6bf8165f5874a663dff0580fc0ab93d9), [842160eb](https://github.com/Savas07/KenshiCoop/commit/842160ebb2d6aaec98e17429be26313ca30f61e0), [876ec41b](https://github.com/Savas07/KenshiCoop/commit/876ec41bf4bf24e632c8210a121d3659f11a9ed8), [58472712](https://github.com/Savas07/KenshiCoop/commit/584727121a5e547fb0a9fce2d75e0b43695913e5), [5cac89bf](https://github.com/Savas07/KenshiCoop/commit/5cac89bf9ebbdda81add81a9fe1168177744fed6), [3b1ad984](https://github.com/Savas07/KenshiCoop/commit/3b1ad984c901fd72b31f537dad4c992041927155), [77026f44](https://github.com/Savas07/KenshiCoop/commit/77026f447d7f1c7d0d5fafd090890b97effd3e58), [47e99af0](https://github.com/Savas07/KenshiCoop/commit/47e99af0cfec0482dd9c454eead24651b49a5c53), [ded25a75](https://github.com/Savas07/KenshiCoop/commit/ded25a75406ea5a4549d1c8357efabfe42b91b89), [50fcd027](https://github.com/Savas07/KenshiCoop/commit/50fcd027c95a8bea6dde7bc07fba9cf7b898e60c), [aa74a2b9](https://github.com/Savas07/KenshiCoop/commit/aa74a2b95041cbc0f1792a1b943784a7c56b3b33), [4b60185d](https://github.com/Savas07/KenshiCoop/commit/4b60185d3a2651463a13412c132fe87c52d5bdda), [5e781654](https://github.com/Savas07/KenshiCoop/commit/5e7816542e590e89301bdf1a0e6a993b0b9849d5).
- Files: `scripts/scenarios.psd1`, `scripts/oracles/Combat.ps1`, `scripts/oracles/Inventory.ps1`, `scripts/oracles/World.ps1`, `src/plugin/test/ScenarioCombat.cpp`, `src/plugin/test/ScenarioInventory.cpp`, `src/plugin/game/EngineProbe.cpp`, `src/prototest/main.cpp`, `docs/REPLICATION_PITFALLS.md`, `dist/mod-kit`, `dist/mods`, `tools/MultiplayerStartGen/Program.cs`.
- Symbols: `Test-HostPcFight`, `Test-RecruitJoin`, `Test-VendorStock`, `Test-MineYield`, `Test-MachineSingle`, `KENSHICOOP_BANNER`.
- Dependencies: —.

### WO-SNAPSHOT — Complete/incomplete capture semantics, safe empty nested bag reconciliation

- Planned action: `manual_port`.
- Source commits: [b0773ae4](https://github.com/Wo-o-o/KenshiCooop/commit/b0773ae491f3a8428162c4a431a0bab8949ef754).
- Files: `src/plugin/game/EngineInventory.cpp`.
- Symbols: `readInvItems`, `applyContainerContents`, `NESTED-EMPTY-DEFER`.
- Dependencies: `SAV-IDENTITY-BAGS`.
- Integration notes: Keep Savas allowContainerCreate/outContainerShort signatures and bag grace. Reject malformed parent graph before mutation.

### WO-LATCH — Graph-safe parent remap and accepted-transfer protection until snapshot catchup

- Planned action: `manual_port`.
- Source commits: [975d2dbe](https://github.com/Wo-o-o/KenshiCooop/commit/975d2dbe8346ecfadfb8f1c6561cdc8d25290a3f).
- Files: `src/netproto/InventorySnapshot.h`, `src/plugin/sync/ReplicatorItems.cpp`.
- Symbols: `compactInventorySnapshot`, `applyInventories`, `applyXferAcks`, `adjustedTruncated`, `localTruncated`.
- Dependencies: `WO-SNAPSHOT`.
- Integration notes: Retain accepted ACK latch; rejected/partial verdicts still release. Preserve shop/research/corpse rebase variants.

### WO-WORLD — Consumed proxy replay tombstones, ownership filter; structural baseline overlaps Savas

- Planned action: `manual_port_unique_hunks`.
- Source commits: [3deedd1f](https://github.com/Wo-o-o/KenshiCooop/commit/3deedd1ff5efde972042d8a7a5c3045846842773).
- Files: `src/plugin/game/EngineInventory.cpp`, `src/plugin/sync/ReplicatorItems.cpp`, `src/plugin/sync/Replicator.h`, `src/plugin/sync/ReplicatorCore.cpp`.
- Symbols: `consumedWorldItems_`, `captureWorldItems`, `publishWorldItems`, `applyWorldItems`, `resetSession`.
- Dependencies: `SAV-WORLD`.
- Integration notes: Keep Savas pending/dormant drops and NativeTaken; add tombstones, nofac-aware ownership filter and peer-owner check before unresolved-drop bookkeeping. No whole-commit exact patch-id duplicate in Savas.

### WO-ABORT — Notify sender failure, cancel receiver and discard stale save/load work on actual disconnect

- Planned action: `manual_port`.
- Source commits: [1125031c](https://github.com/Wo-o-o/KenshiCooop/commit/1125031ca81ca6de880159979b75cd46110ae20b).
- Files: `src/plugin/sync/SaveXfer.cpp`, `src/plugin/sync/SaveXfer.h`, `src/plugin/core/Inbound.h`, `src/plugin/Plugin.cpp`.
- Symbols: `sendAbortAndNotify`, `abortReceive`, `Inbound::discardSaveLoadState`, `sessionResetForUi`, `processNetEvents`.
- Dependencies: `SAV-SAVE`, `SAV-NET8`.
- Integration notes: Savas has no corresponding abort/flush helper. Sender failure DONE must use Savas g_sendMask; one join departure must not abort another join transfer.

### WO-CI — Portable mocked regressions and explicit genuine v100 integration gate

- Planned action: `port_and_adapt_tests`.
- Source commits: [15c25077](https://github.com/Wo-o-o/KenshiCooop/commit/15c25077202848d4283bc362c479776a4f1fa528), [c3c10045](https://github.com/Wo-o-o/KenshiCooop/commit/c3c1004543e19a4b03afe30405ba798f67832455).
- Files: `.github/workflows/stabilization.yml`, `scripts/tests/portable_regressions.py`, `scripts/tests/portable`, `scripts/tests/check_re_integration.py`, `scripts/build_snapshot_tests.cmd`, `docs/TECHNICAL_AUDIT_V051.md`, `docs/RE_KENSHI_INTEGRATION.md`.
- Symbols: `InventoryRegression`, `SnapshotRegression`, `WorldRegression`, `SaveAbortRegression`.
- Dependencies: `WO-SNAPSHOT`, `WO-LATCH`, `WO-WORLD`, `WO-ABORT`.
- Integration notes: Mocks target v0.51 APIs and need adaptation; portable tests cannot prove v100 game ABI or live gameplay.

### XIM-TRADER — Cross-machine merchant descriptor pairing and deferred release stock table

- Planned action: `cherry_pick_then_lifecycle_fix`.
- Source commits: [88f0dd94](https://github.com/ximery4/KenshiCoop/commit/88f0dd940c471fcdf082aae9544fcd7d7cc8fa61), [13931f29](https://github.com/ximery4/KenshiCoop/commit/13931f29ee3b6f0874ffcec3647416b457a748ed).
- Files: `src/netproto/Wire.h`, `src/plugin/sync/Replicator.h`, `src/plugin/sync/ReplicatorItems.cpp`, `src/prototest/main.cpp`.
- Symbols: `TRADE_FLAG_DESC`, `TraderDesc`, `TradePark`, `traderLocalKey`, `describeTraderKey`, `handleTradeOp`, `tickTradeLocal`, `KENSHICOOP_TRADE_SCRAMBLE`.
- Dependencies: `SAV-TRADER`.
- Integration notes: Add resetSession/peer-leave clearing for 4 new containers, protocol unification, match ambiguity tests. Old trader-identity has orphan snapshot history and should not be merged wholesale.

### XIM-XFER — Fabricate first-take fixture shortfall while excluding corpses and owned bags

- Planned action: `take_once_experimental_manual_port`.
- Source commits: [5f89ce8c](https://github.com/ximery4/KenshiCoop/commit/5f89ce8c7e8d03fdfaa25acae53f52e9aeb615ce), [d253e787](https://github.com/ximery4/KenshiCoop/commit/d253e78704bf924a4798d94617bd13643f7cc4c1), [6e62cebc](https://github.com/ximery4/KenshiCoop/commit/6e62cebc35dde0440046d63c400098809f3c2203).
- Files: `src/plugin/sync/ReplicatorItems.cpp`.
- Symbols: `applyTransfers`, `KENSHICOOP_XFER_CENSUS_FAB`, `srcCensusOnly`.
- Dependencies: `SAV-XFER-RACE`, `SAV-PRODUCTION`.
- Integration notes: 5f89ce8 tree equals d253e78+6e62ceb full tree. Do not take both. Default-on bypass can also fabricate an empty shared world chest; require source provenance and chest/corpse race tests; default off until proved.

### XIM-DIAG — Fabrication failure reason instrumentation unique to old orphan branch

- Planned action: `manual_port_optional_diagnostics`.
- Source commits: [9eedc089](https://github.com/ximery4/KenshiCoop/commit/9eedc089978f5af4a9a857f0f9a7e0c01d1008ed).
- Files: `src/plugin/game/Engine.h`, `src/plugin/game/EngineInventory.cpp`, `src/plugin/sync/ReplicatorItems.cpp`.
- Symbols: `lastAddItemsFailReason`, `g_addFailReason`, `fabWhy`.
- Dependencies: `XIM-XFER`.
- Integration notes: Not present in two clean upstream patch branches; preserve if goal covers every unique instrumentation change.

### CAP-REKEY — Session-lifetime serial-checked local-to-wire map for census after stream grace expires

- Planned action: `manual_port_not_already_fully_covered`.
- Source commits: [fc90aff7](https://github.com/captainjoshua77-wq/KenshiCoop/commit/fc90aff7a7250a6b72ad98f68075ea9e5d55f4ed).
- Files: `ci/fix.patch`, `src/plugin/sync/Replicator.h`, `src/plugin/sync/ReplicatorAuthority.cpp`, `src/plugin/sync/ReplicatorCore.cpp`, `src/plugin/sync/ReplicatorDrive.cpp`.
- Symbols: `wireOfLocal_`, `applyTargets`, `enforceHostAuthority`, `resetSession`, `pruneDriveGrace`.
- Dependencies: `SAV-IDENTITY-BAGS`.
- Integration notes: Repository commit only stores patch. Savas canonicalOf_/identityKeyOf overlaps but canonicalOf_ prunes after 30 seconds; patch persists reverse map until reset. Need validate map invalidation/rekey and leave.

### CAP-DOOR — Standalone placed-door wire/local hand translation with echo baseline

- Planned action: `manual_port`.
- Source commits: [fc90aff7](https://github.com/captainjoshua77-wq/KenshiCoop/commit/fc90aff7a7250a6b72ad98f68075ea9e5d55f4ed).
- Files: `ci/fix.patch`, `src/plugin/sync/ReplicatorChannels.cpp`.
- Symbols: `publishDoors`, `applyDoors`, `mintByLocal_`, `peerBuilds_`, `doorRows_`.
- Dependencies: —.
- Integration notes: No packet layout change; specifically standalone door/gate without parent building; test both placer directions and no echo.

### COOK-NOTES — Fork notes and local state ignore only

- Planned action: `docs_optional_preserve_provenance`.
- Source commits: [768d25be](https://github.com/cookingwithsimon/KenshiCoop/commit/768d25bec4f8be45bb64f0a51205a8626bb31a9e), [0c493a94](https://github.com/cookingwithsimon/KenshiCoop/commit/0c493a94229cda1397c7bffe659503e905bdaa61).
- Files: `.gitignore`, `NOTES.md`.
- Symbols: `local/ ignore`.
- Dependencies: —.

## Detailed port requirements

### Final Savas behavior to preserve

The final base includes safeguards that smaller donor patches must retain:

- SAV-CHARGEN transports native appearance/name/race through CharGenHeader and only admits a fresh New Game. WIP `2e1f5ce` alone does not build; preserve the completed path.
- SAV-ROSTER persists Steam identities through kenshicoop_players.txt, named tabs and absent-player substitution. Alternative ownership rules need one policy, not competing assignments.
- SAV-SAVE uses Unicode Win32 APIs, avoids overwriting existing bootstrap saves, orders BEGIN/chunk/DONE, rejects unsafe names/paths, requires quick.save, verifies CRC/seen files and commits __incoming→final through __old with stranded-save recovery. WO-ABORT adds lifecycle cancellation rather than replacing these checks.
- SAV-DEVTOOLS is opt-in: overlay, recorder, watchdog/dumps, UDP telemetry, dashboard, replay and scripted move reproduction. The master switch defaults off; replay is a separate tool.
- SAV-VENDOR counter/delta/cash paths and SAV-TRADER window handover are different layers. SAV-PRODUCTION guards production/crafting specifically; do not indiscriminately guard every Building::operate call.
- SAV-RECRUIT keeps correct ownership, wire-space subjects, owner pins, excludes squad bodies from world-NPC publication and clears stale AI walking.
- SAV-COMBAT resolves subjects through body mappings and uses a per-owner-target reissue budget. Historical opposite re-order commits are not two desired policies; preserve final ef40acd behavior.
- SAV-WORLD has a structural baseline, unresolved/dormant own drops and NativeTaken notices. Keep these rather than an older temporal baseline. The function is applyNativeTakes.
- SAV-RESEARCH needs both grow-only technology union and authoritative book costs. SAV-JOINSTATE carries names/jobs with per-sender/kind/key sequences and translated building targets; preserve its directions and host policy.
- SAV-XFER-RACE treats authoritative source shortfall as already-taken stock and reconciles optimistic losers through rejected/partial ACK. XIM-XFER changes this invariant for selected sources and therefore requires conservation tests.
- SAV-IDENTITY-BAGS covers more than backpack creation: creation after WD_REHOME_MAX_MS+4000 while preserving W2 relocation grace; indexProxies/wireKeyOf/publishKeyFor/pruneLastHands/xlateSubjects; stable NPC identity after squad splits; medical/treatment/combat/carry translation; cell-claim hysteresis; no freezing driven proxies or parking carried bodies; and a first-1000-ms dialogue pause that must not become a global speed vote.
- Rebuild DLLs and generate .mod/2–8-player starts from one final commit; do not combine binaries from different dist directories.

### Wo snapshots, latches and world items

WO-SNAPSHOT marks missing/failed g_getSectionsFn capture incomplete, continues searching for omitted nested entries after the output fills, and validates parent range/self-parent/nested-parent constraints before mutation. A complete parent bag without children means empty; a truncated snapshot does not authorize clearing unseen children. Ambiguous identical bags or truncated parent recapture produce NESTED-EMPTY-DEFER. Preserve Savas allowContainerCreate/outContainerShort and creation grace during the manual port.

WO-LATCH compacts tombstones after adjustments, remaps surviving parentIdx values, removes children of removed parents and defers invalid/overflowed graphs. Catch-up requires complete local and remote snapshots; synthetic backpack parents make a snapshot additive-only. ACCEPT acknowledges a mutation, not a fresh snapshot, so keep the accepted latch until catch-up/deadline while rejected/partial outcomes release for rollback. Preserve Savas shop/research/corpse rebase branches and bagShortMs. Audit local-versus-desired totals at the same nesting level; this is a port check, not a reproduced donor bug.

WO-WORLD overlaps Savas structural baseline/pending drops, but adds unique consumedWorldItems_ receive tombstones keyed by (ownerId, netId), reset cleanup, a peer-owner filter before unresolved-drop bookkeeping and a non-player ownership filter with the nofac sentinel exception. Savas worldTrack.claimed prevents repeated author-side CLAIM, not the same receive replay window. Preserve eight-player scoping, NativeTaken, dormant/native distinctions and pointer guards. Tombstones belong to a world epoch; test sender/netId reuse after reconnect.

### Wo save cancellation and CI

WO-ABORT sends a zero-file SaveDoneHeader on sender open/read failure. Savas already safely rejects empty DONE, but its sender failure only aborts locally. abortReceive closes handles, ends active receive, retains staging for diagnosis and advances result/sequence; abortAll includes the receiver. Clear queued BEGIN/FILE/DONE/ACK/LOAD GO/REQ/NACK on actual disconnect, not indiscriminately on a connected world reload.

Do not copy two-player “any leave cancels everything” into an eight-player host. Preserve other recipients’ pending work and use g_sendMask/target identity for failure notification. Cancellation should be scoped by (sessionEpoch, peerId, transferId); a join losing the host can cancel its entire receive session.

WO-CI provides InventoryRegression, SnapshotRegression, WorldRegression and SaveAbortRegression mocks, sanitizer runner, prototest, check_re_integration.py, snapshot tests and a genuine Windows v100 gate. The prepared-runner job requires explicit dispatch; passing portable mocks does not establish Kenshi ABI compatibility. Adapt signatures/multi-peer cancellation, retain documentation attribution and rerun on the integration build.

### Ximery merchant pairing and lifecycle

Clean upstream/trader-identity is based exactly on Savas585: 88f0dd9 changes Wire.h, Replicator.h/Items.cpp and prototest; 13931f2 changes protocol to 165. TradeHeader adds TRADE_FLAG_DESC, tSid[48], tFac[48] and position. traderLocalKey tries a cached pair, then raw-hand template/position, then adoptCandidateNear within 20 units. Unmatched traders receive a foreign high-bit, sender-namespaced key. Deferred join RELEASE stock retries each second for up to ten minutes, bounded to 64 records. handleTradeOp/tickTradeLocal split the lock logic; KENSHICOOP_TRADE_SCRAMBLE exercises cross-machine mapping under Harness.

Mandatory corrections before enablement:

- Clear tradePeerKey_, tradeDesc_, tradePark_ and tradeMissLog_ on reset and selectively on peer departure; tag them with epoch. Their original usages contain no resetSession cleanup. Old parked stock could match a trader in a new world: a static risk, not a reproduced failure.
- Cache resolution alone does not validate descriptor/epoch. The raw-hand fast path checks SID/distance but not faction; nearest-candidate matching does not prove uniqueness. Test recycled hands, identical nearby traders, vertical separation, moving traders and unload/reload; reject ambiguous matches.
- Protocol 165 is only a donor branch identifier. The integration requires one schema/layout/dispatch specification and explicit mismatch rejection.

### Ximery fixture fabrication and diagnostics

Final 5f89ce8 changes applyTransfers and its diagnostics; its complete tree equals d253e78+6e62ceb. srcCensusOnly permits fabrication even when sAuth==localId if moved==0, sk.t==0, source is in censusContainers_ and not censusCorpses_/ownHands_/ownedContainers_. The donor flag KENSHICOOP_XFER_CENSUS_FAB defaults on; zero restores the old rule. The earlier broad form included corpses and was narrowed after an author-reported duplicate-loot problem.

Preserve the final form as an experimental candidate, not an automatically enabled integration fix. Type 0 includes shared chests/boxes; local absence does not prove the item was never taken. Require proven production-source provenance or author output transaction with an idempotent token. Test concurrent chest emptying, corpse races, first campfire/node yield, replayed ACK/transferId, partial quantities and reconnect. Keep default-off or diagnostics-only until established.

Old orphan trader-identity uniquely contains 9eedc08 lastAddItemsFailReason/g_addFailReason and fabWhy, absent from clean upstream/* branches. Port that small diagnostic independently. Reasons include bad arguments, unresolved container hand, missing inventory, item/template creation failure and skipped work. Its global static requires main-thread-only calls. Old snapshot packaging/gitlinks are not new gameplay features.

### Captain patch: rekey and standalone doors

Repository commit fc90aff stores a five-file patch with 64 insertions/3 deletions. A temporary-index git apply --cached --check passes on upstream but not the full Savas patch context. The standalone ReplicatorChannels door hunks separately pass that mechanical check on Savas. Mechanical applicability is not runtime validation.

CAP-REKEY retains a serial-validated local-to-wire reverse map through the session, so a wide census does not lose translation after drive grace expires. Savas identityKeyOf/canonicalOf_ overlaps, but pruneDriveGrace removes canonicalOf_ after 30 seconds. Preserve the long-tail invariant while auditing destruction, rekeys, serial reuse, reset and peer departure.

CAP-DOOR translates parentless placed-door/gate identities through mintByLocal_/peerBuilds_ and updates doorRows_ to prevent echo. No packet-layout change is needed. Test both placer directions, standalone versus parented doors and absence of echo.

## Integration order and future checks

Follow the [consolidated plan](../../MIGRATION_PLAN.md): pin contracts/epochs → save cancellation → stable identity → complete snapshots/latches/world-item conservation → intents/combat → multi-peer validation → optional policies → UI/backends/release. Keep opt-in diagnostics and attribution. Every newly introduced queue/map/cancel must survive host plus at least two joins, selective leave/rejoin and peer-ID reuse.

Future gates include genuine v100 Harness/Release, prototest packet/queue contracts, adapted portable regressions, live scenarios from both lines, two physical PCs for differing engine hands/trader mapping, at least three-player leave/rejoin/save, and reload with bags/jobs/roster. These are planned, not executed.

## Complete feature index

| ID | Action | Source SHAs | Dependencies |
|---|---|---|---|
| SAV-NET8 | included_base | a5b12518 | — |
| SAV-LOBBY | included_base | a5b12518, 6df1084e, d7a3cc07, de1d5c22, aaea80a6 | SAV-NET8 |
| SAV-CHARGEN | included_base | 2e1f5ce4, 433f3275, 6fb51c09, de1d5c22, de302373 | SAV-NET8, SAV-SAVE |
| SAV-ROSTER | included_base | be1a0519, 3a567b92, 29090d90 | SAV-NET8 |
| SAV-SAVE | included_base | 88ec9518, 6fb51c09, de1d5c22, 0c90ef83 | — |
| SAV-DRIVE | included_base | 433f3275, ec897642, 86b95e08, 4adca37c, fa71c222 | SAV-NET8 |
| SAV-DEVTOOLS | included_base_opt_in | 4f185bcc, fff395ae | — |
| SAV-VENDOR | included_base | 3198745f, 19e2f625, ebbc6f0e, e6e1ddde, 7c6e19c3 | SAV-NET8 |
| SAV-PRODUCTION | included_base | e5ad2ef0, f80779ed | SAV-NET8 |
| SAV-RECRUIT | included_base | 0b8f63f8, 3e9bd90b, 5c5f3886 | SAV-NET8 |
| SAV-COMBAT | included_base | 337004a2, 5d845833, 65b81d53, d2509ed3, ef40acd5 | SAV-DRIVE |
| SAV-WORLD | included_base | d82a496d | SAV-NET8 |
| SAV-CORPSE | included_base | 0c3def37 | SAV-VENDOR |
| SAV-RESEARCH | included_base | a6c97be0 | SAV-VENDOR |
| SAV-JOINSTATE | included_base | 76975155 | SAV-NET8, SAV-ROSTER |
| SAV-XFER-RACE | included_base | 32440706 | SAV-CORPSE |
| SAV-GEAR | included_base | 4bd312f8 | — |
| SAV-TRADER | included_base | 19497e70, 3ec2551a | SAV-VENDOR |
| SAV-IDENTITY-BAGS | included_base | 58514ea3 | SAV-RECRUIT, SAV-DRIVE, SAV-COMBAT |
| SAV-SUPPORT | included_base_rebuild_artifacts | 1c147364, c939bcce, f76021a4, 79e4645f, 73518a01, 74049b22, 01f9d227, eecb4e2e, 842160eb, 876ec41b, 58472712, 5cac89bf, 3b1ad984, 77026f44, 47e99af0, ded25a75, 50fcd027, aa74a2b9, 4b60185d, 5e781654 | — |
| WO-SNAPSHOT | manual_port | b0773ae4 | SAV-IDENTITY-BAGS |
| WO-LATCH | manual_port | 975d2dbe | WO-SNAPSHOT |
| WO-WORLD | manual_port_unique_hunks | 3deedd1f | SAV-WORLD |
| WO-ABORT | manual_port | 1125031c | SAV-SAVE, SAV-NET8 |
| WO-CI | port_and_adapt_tests | 15c25077, c3c10045 | WO-SNAPSHOT, WO-LATCH, WO-WORLD, WO-ABORT |
| XIM-TRADER | cherry_pick_then_lifecycle_fix | 88f0dd94, 13931f29 | SAV-TRADER |
| XIM-XFER | take_once_experimental_manual_port | 5f89ce8c, d253e787, 6e62cebc | SAV-XFER-RACE, SAV-PRODUCTION |
| XIM-DIAG | manual_port_optional_diagnostics | 9eedc089 | XIM-XFER |
| CAP-REKEY | manual_port_not_already_fully_covered | fc90aff7 | SAV-IDENTITY-BAGS |
| CAP-DOOR | manual_port | fc90aff7 | — |
| COOK-NOTES | docs_optional_preserve_provenance | 768d25be, 0c493a94 | — |

## Coverage of all 73 Savas commits

| SHA | Commit date | Subject | Feature IDs |
|---|---|---|---|
| [a5b12518](https://github.com/Savas07/KenshiCoop/commit/a5b1251891bb7022a33cc09652aad88daacc7863) | 2026-10-17T21:37:22+03:00 | coop: up to 8 players + one-click Steam invites (protocol 56) | SAV-NET8, SAV-LOBBY |
| [6df1084e](https://github.com/Savas07/KenshiCoop/commit/6df1084e8f99bdc28b3379f84b0cb43d759681f3) | 2026-10-17T21:58:34+03:00 | coop: friends wait in the lobby until the host's world is live, then connect | SAV-LOBBY |
| [d7a3cc07](https://github.com/Savas07/KenshiCoop/commit/d7a3cc07c6362a79e73a999d4d13c6965bd53b51) | 2026-10-17T22:12:44+03:00 | coop: a join with no local saves can enter the host's world | SAV-LOBBY |
| [1c147364](https://github.com/Savas07/KenshiCoop/commit/1c147364c5886dd0ef8cd99d0d6efe1608e7b270) | 2026-10-17T22:22:00+03:00 | coop: joining players log as [JOIN]; kit README says where the log is | SAV-SUPPORT |
| [88ec9518](https://github.com/Savas07/KenshiCoop/commit/88ec95183e74c9d6977d660def6e87f318cca48d) | 2026-10-17T22:54:52+03:00 | savexfer: Unicode file APIs so translated saves transfer | SAV-SAVE |
| [2e1f5ce4](https://github.com/Savas07/KenshiCoop/commit/2e1f5ce421272a899950c72936a19715110394c8) | 2026-10-18T00:15:13+03:00 | WIP v0.55: version constant, MoveSync partial (does not build yet) | SAV-CHARGEN |
| [433f3275](https://github.com/Savas07/KenshiCoop/commit/433f3275ed7c46a2a4175cd129b4182ffdb51d94) | 2026-10-18T02:02:54+03:00 | v0.55: guest character creation, smoother remote movement, Steam crash guard | SAV-CHARGEN, SAV-DRIVE |
| [6fb51c09](https://github.com/Savas07/KenshiCoop/commit/6fb51c092b584bc7cbd34718fe50cfac9f162e35) | 2026-10-18T02:28:24+03:00 | v0.55.1: character creation only on worlds the host started with New Game; save-transfer DONE/chunk race fix | SAV-CHARGEN, SAV-SAVE |
| [de1d5c22](https://github.com/Savas07/KenshiCoop/commit/de1d5c22dbf247d52d79410a2145595191c006e4) | 2026-10-18T07:08:30+03:00 | v0.56: main-menu MULTIPLAYER lobby; new-game connect push no longer overwrites a save | SAV-LOBBY, SAV-CHARGEN, SAV-SAVE |
| [de302373](https://github.com/Savas07/KenshiCoop/commit/de3023734003a35c62d3a74db750481e128a70c7) | 2026-10-18T07:24:45+03:00 | v0.56.1: track New Game freshness on every world-live edge, whatever the role | SAV-CHARGEN |
| [aaea80a6](https://github.com/Savas07/KenshiCoop/commit/aaea80a6ec3f12bdb408d5ba1c6b7b05015e3474) | 2026-10-18T16:04:22+03:00 | v0.56.2: Steam rich presence for joining + invite/join diagnostics | SAV-LOBBY |
| [be1a0519](https://github.com/Savas07/KenshiCoop/commit/be1a05193b8578b683a8dc704f341dafd31e6618) | 2026-10-18T17:26:25+03:00 | v0.57: squad tabs named after their players; who-had-which-squad remembered in the save | SAV-ROSTER |
| [3a567b92](https://github.com/Savas07/KenshiCoop/commit/3a567b9259017bddbe7c11a4fd6f65055e7caa6a) | 2026-10-18T17:37:46+03:00 | v0.57.1: a substitute player takes the absent player's squad | SAV-ROSTER |
| [29090d90](https://github.com/Savas07/KenshiCoop/commit/29090d908778ad775068dd3d210c8cd3a908af10) | 2026-10-18T18:07:00+03:00 | v0.57.2: squad tab bar shows the player names at once; status texts point to MULTIPLAYER | SAV-ROSTER |
| [5e781654](https://github.com/Savas07/KenshiCoop/commit/5e7816542e590e89301bdf1a0e6a993b0b9849d5) | 2026-10-18T19:08:12+03:00 | v0.57.3: hide the top-left status banner (KENSHICOOP_BANNER=1 shows it for debugging) | SAV-SUPPORT |
| [ec897642](https://github.com/Savas07/KenshiCoop/commit/ec89764294f50e1ded464c015acb828bd2ce783c) | 2026-10-18T23:26:28+03:00 | v0.57.4-test (local, NOT released): squad glide + get-up release | SAV-DRIVE |
| [86b95e08](https://github.com/Savas07/KenshiCoop/commit/86b95e08f866277ec3e4e0740956b6dee384b9f8) | 2026-10-19T00:57:42+03:00 | v0.57.4-test2 (local, NOT released): glide keeps the model and the gait | SAV-DRIVE |
| [4adca37c](https://github.com/Savas07/KenshiCoop/commit/4adca37c471e1a3e0382393b52818d596b773567) | 2026-10-19T01:24:10+03:00 | v0.57.4-test3 (local, NOT released): never glide a carrier | SAV-DRIVE |
| [4f185bcc](https://github.com/Savas07/KenshiCoop/commit/4f185bcc20e877b06988127ff042658f551422d3) | 2026-10-19T16:02:31+03:00 | dev/tools (local, NOT released): developer tools for co-op sync debugging | SAV-DEVTOOLS |
| [fff395ae](https://github.com/Savas07/KenshiCoop/commit/fff395aeaa49b740c10fc6df6d4e8491ad5f03a3) | 2026-10-19T22:43:10+03:00 | dev/tools: move <hand> <dx> <dz> command for scripted repros | SAV-DEVTOOLS |
| [c939bcce](https://github.com/Savas07/KenshiCoop/commit/c939bcce42aa0d77b9341c032efe216b8156b3de) | 2026-10-19T22:45:20+03:00 | v0.57.5: no more teleports/frozen models/animationless sliding for friends' characters; carried bodies stay visible; dev tools (off by default) | SAV-SUPPORT |
| [f76021a4](https://github.com/Savas07/KenshiCoop/commit/f76021a42376e30c485202c0c6f15f575421fd88) | 2026-10-20T15:01:13+03:00 | harness: host_pc_fight(+_mint) scenario + Test-HostPcFight oracle (join's copy of a fighting host PC) | SAV-SUPPORT |
| [79e4645f](https://github.com/Savas07/KenshiCoop/commit/79e4645f1198a2e6275b46484e7d7a1c21a8511a) | 2026-10-20T15:05:36+03:00 | recruit_join: scenario + Test-RecruitJoin oracle (join recruits a host-driven NPC; host-recruit control arm) | SAV-SUPPORT |
| [73518a01](https://github.com/Savas07/KenshiCoop/commit/73518a017e1b3b1672b3e12d0b5adbfb7218d815) | 2026-10-20T15:38:42+03:00 | harness: host_pc_fight - buffed fighters, fight after the join clock catch-up; gate on engaged-with-owner's-target | SAV-SUPPORT |
| [3198745f](https://github.com/Savas07/KenshiCoop/commit/3198745f76e67edf6a64f9dbe68eefcd6e636075) | 2026-10-20T15:40:00+03:00 | vendor stock: shared shop counters (protocol 60) + vendor_stock scenario | SAV-VENDOR |
| [74049b22](https://github.com/Savas07/KenshiCoop/commit/74049b224724df52777184beeb947f7ea14c958f) | 2026-10-20T16:16:48+03:00 | harness: mine_yield / mine_yield_join / bench_yield_join yield scenarios + Test-MineYield, Test-MachineSingle oracles | SAV-SUPPORT |
| [e5ad2ef0](https://github.com/Savas07/KenshiCoop/commit/e5ad2ef0a85fdecd88bbd9bb8b96d9c58fcec4ba) | 2026-10-20T16:16:48+03:00 | fix: machine output simulated once (host); join skips operate() for driven workers and host-mirrored machines; [prod] operate diagnostics | SAV-PRODUCTION |
| [0b8f63f8](https://github.com/Savas07/KenshiCoop/commit/0b8f63f8de94288ed12cf5e13738ab6007b1c416) | 2026-10-20T16:21:01+03:00 | recruit: a join recruit of a host-driven NPC is the join's on both games | SAV-RECRUIT |
| [01f9d227](https://github.com/Savas07/KenshiCoop/commit/01f9d22711386f4a8911a09059a5539313040150) | 2026-10-20T16:22:55+03:00 | docs: pitfall 19 - name an identity edge's subject in the receiver's key space (recruit_join) | SAV-SUPPORT |
| [eecb4e2e](https://github.com/Savas07/KenshiCoop/commit/eecb4e2e6bf8165f5874a663dff0580fc0ab93d9) | 2026-10-20T16:23:55+03:00 | Test-VendorStock: take the earliest purchase time without Measure-Object -Property on hashtables | SAV-SUPPORT |
| [842160eb](https://github.com/Savas07/KenshiCoop/commit/842160ebb2d6aaec98e17429be26313ca30f61e0) | 2026-10-20T16:27:13+03:00 | harness: host_pc_fight resident arm buffs defensive stats only (all-stat leaders chased fleeing bar NPCs at 120 u/s) | SAV-SUPPORT |
| [876ec41b](https://github.com/Savas07/KenshiCoop/commit/876ec41bf4bf24e632c8210a121d3659f11a9ed8) | 2026-10-20T17:08:45+03:00 | harness: host_pc_fight resident arm - host orders each leader's bar victim onto it, drops victims that flee (chases were 70-120 u/s sprints, not melee) | SAV-SUPPORT |
| [3e9bd90b](https://github.com/Savas07/KenshiCoop/commit/3e9bd90b66188bd99ac08e8d873389f2b9bab86a) | 2026-10-20T17:17:37+03:00 | publishOwned: never stream a pinned squad hand as a world NPC (a just-inserted peer recruit passed captureNpcs' playerCharacters filter for one tick; the recruiter REQ'd its fresh hand and minted a phantom twin into its own squad - rj_after2/rj_after_wan) | SAV-RECRUIT |
| [58472712](https://github.com/Savas07/KenshiCoop/commit/584727121a5e547fb0a9fce2d75e0b43695913e5) | 2026-10-20T17:18:19+03:00 | vendor_stock: sell leg (host + join sell one stack back) and merchant-cash gate | SAV-SUPPORT |
| [19e2f625](https://github.com/Savas07/KenshiCoop/commit/19e2f62546cea77308515c6d557f9d4860e5f7f6) | 2026-10-20T17:22:38+03:00 | vendor stock: host-authoritative merchant cash (protocol 60) | SAV-VENDOR |
| [337004a2](https://github.com/Savas07/KenshiCoop/commit/337004a2873dfb1e0f8077af596f0799d447e44b) | 2026-10-20T17:25:51+03:00 | combat: resolve the streamed target through the drive's wire->local translation; judge wrong-target on bodies; no walk-order on arming/in-band squad copies | SAV-COMBAT |
| [5d845833](https://github.com/Savas07/KenshiCoop/commit/5d845833c0baedf62aade21fdba85be3c856072c) | 2026-10-20T17:29:41+03:00 | combat: squad copies get a fresh re-issue budget per owner target and are re-ordered right after a snap | SAV-COMBAT |
| [5cac89bf](https://github.com/Savas07/KenshiCoop/commit/5cac89bf9ebbdda81add81a9fe1168177744fed6) | 2026-10-20T17:39:38+03:00 | recruit_seated: seated arms of recruit_join (both sides recruit a posed bar NPC); oracle gates posed subject + recruiter phantom twin | SAV-SUPPORT |
| [fa71c222](https://github.com/Savas07/KenshiCoop/commit/fa71c2220ace43c7b6c5ba8021ad76caa6c17be5) | 2026-10-20T17:39:39+03:00 | drive: end a squad copy's stale seat/work pose once its owner walks with no task ([heal] POSE) - the peer's copy of a seated recruit sat frozen or crawled | SAV-DRIVE |
| [ebbc6f0e](https://github.com/Savas07/KenshiCoop/commit/ebbc6f0eaee3d818d9e6c4292c4fa7b8ed3a6337) | 2026-10-20T17:52:27+03:00 | vendorBuyInto: place a traded stack exactly once (loose section first) | SAV-VENDOR |
| [65b81d53](https://github.com/Savas07/KenshiCoop/commit/65b81d53996b2e87b1a214952d4d20e0cdf98f7c) | 2026-10-20T18:13:12+03:00 | combat: drop the immediate re-order after a squad-copy snap | SAV-COMBAT |
| [d2509ed3](https://github.com/Savas07/KenshiCoop/commit/d2509ed32ac7dc24f0a394374ba8f1445b115af3) | 2026-10-20T18:25:59+03:00 | combat: restore the immediate re-order after a squad-copy snap | SAV-COMBAT |
| [3b1ad984](https://github.com/Savas07/KenshiCoop/commit/3b1ad984c901fd72b31f537dad4c992041927155) | 2026-10-20T18:26:18+03:00 | docs: pitfalls 19 - squad-copy walk orders and per-target re-issue budget | SAV-SUPPORT |
| [77026f44](https://github.com/Savas07/KenshiCoop/commit/77026f447d7f1c7d0d5fafd090890b97effd3e58) | 2026-10-20T18:58:26+03:00 | WIP fix/combat: host_pc_fight gate not yet stable after the fix | SAV-SUPPORT |
| [47e99af0](https://github.com/Savas07/KenshiCoop/commit/47e99af0cfec0482dd9c454eead24651b49a5c53) | 2026-10-20T19:05:16+03:00 | harness: probe machine templates keyed on stringID (localised names broke placement); template dump; bench_yield_join input seeding, demoted to diagnostic | SAV-SUPPORT |
| [e6e1ddde](https://github.com/Savas07/KenshiCoop/commit/e6e1dddebfd2098722a21aaff4d2cffb4bab1f39) | 2026-10-20T19:39:01+03:00 | vendor_stock: place a traded stack exactly once; sell through the trade window's shop view | SAV-VENDOR |
| [ded25a75](https://github.com/Savas07/KenshiCoop/commit/ded25a75406ea5a4549d1c8357efabfe42b91b89) | 2026-10-20T19:44:50+03:00 | docs: pitfall 19 - receiver-side NPC-SKIP and seat-pose heal follow-ups | SAV-SUPPORT |
| [7c6e19c3](https://github.com/Savas07/KenshiCoop/commit/7c6e19c387035cfb8620b797601f86fdd67819a3) | 2026-10-20T20:01:35+03:00 | vendor_stock: drop the unexercised shop-view sale path; a sale is buyItem + place-once | SAV-VENDOR |
| [ef40acd5](https://github.com/Savas07/KenshiCoop/commit/ef40acd5c3b01f6e3c99ef2f8db42c9de160eaf0) | 2026-10-20T20:36:43+03:00 | combat: an engaged squad copy is never walk-ordered or sustained-snapped mid-brawl | SAV-COMBAT |
| [f80779ed](https://github.com/Savas07/KenshiCoop/commit/f80779edf95b9725890240114c211223422c2f66) | 2026-10-20T21:38:44+03:00 | prod: gate only production+crafting operate on the host; prove crafting with bench_yield_join; re-pose lapsed machine copies | SAV-PRODUCTION |
| [50fcd027](https://github.com/Savas07/KenshiCoop/commit/50fcd027c95a8bea6dde7bc07fba9cf7b898e60c) | 2026-10-20T23:01:57+03:00 | docs: pitfall 19 - engaged lock hold for squad copies; brawl-level engagement gate | SAV-SUPPORT |
| [ee6da702](https://github.com/Savas07/KenshiCoop/commit/ee6da70291117e2495ef7163046bd6803bd52739) | 2026-10-20T23:05:19+03:00 | Merge branch 'fix/recruit' into integ/all | SAV-SUPPORT |
| [03d17404](https://github.com/Savas07/KenshiCoop/commit/03d17404d9ab15b8776f53e89953a419fa1a722b) | 2026-10-20T23:06:52+03:00 | Merge branch 'fix/vendor' into integ/all | SAV-SUPPORT |
| [4a2b69e2](https://github.com/Savas07/KenshiCoop/commit/4a2b69e27596e4bb7849f55dded1e34ef84bbba3) | 2026-10-20T23:09:58+03:00 | Merge branch 'fix/combat' into integ/all | SAV-SUPPORT |
| [51f3e178](https://github.com/Savas07/KenshiCoop/commit/51f3e178f37ceee5f98852c7e16612428cb3a3f8) | 2026-10-20T23:55:42+03:00 | Merge branch 'fix/ore' into integ/all | SAV-SUPPORT |
| [d82a496d](https://github.com/Savas07/KenshiCoop/commit/d82a496d8ac1994f008818394728e82244a174aa) | 2026-10-21T01:22:03+03:00 | world items: save-native ground items are structural baseline + pickup notice (upstream #44 #63 #66) | SAV-WORLD |
| [0c3def37](https://github.com/Savas07/KenshiCoop/commit/0c3def37aca7f58ecd40fb3bc799489428f61fcf) | 2026-10-21T02:12:22+03:00 | fix(sync): replicate corpse inventories so loot is not duplicated (upstream #40) | SAV-CORPSE |
| [a6c97be0](https://github.com/Savas07/KenshiCoop/commit/a6c97be05cddc63fda4f8aade029a67814126cbf) | 2026-10-21T03:24:30+03:00 | Fix #84: research started on a join pays no books and never reaches the host (protocol 63) | SAV-RESEARCH |
| [76975155](https://github.com/Savas07/KenshiCoop/commit/769751554310e1976e42155d4ac81457f9dd02e7) | 2026-10-21T04:31:09+03:00 | Fix #74.1: join squad names, faction name and permanent jobs wiped by the host-authored save (protocol 64) | SAV-JOINSTATE |
| [0c90ef83](https://github.com/Savas07/KenshiCoop/commit/0c90ef83f1c7dcd1b38f04ac28d664afe18bb07e) | 2026-10-21T04:31:51+03:00 | save: commit a transferred save only when it is complete, loadable and safely named (upstream #16) | SAV-SAVE |
| [32440706](https://github.com/Savas07/KenshiCoop/commit/32440706db1c86789dc467608ad7b9a4c61c3f6f) | 2026-10-21T15:36:45+03:00 | inventory: refuse a loot of a unit that already left a container we author, and roll the loser's copy back (upstream #40 race) | SAV-XFER-RACE |
| [4bd312f8](https://github.com/Savas07/KenshiCoop/commit/4bd312f8b9d597a57024afebc6955cc750404668) | 2026-10-21T16:54:33+03:00 | inventory: name the gear object that actually left the bag, and see a drop that empties the character (manual T8) | SAV-GEAR |
| [5c5f3886](https://github.com/Savas07/KenshiCoop/commit/5c5f3886f93769a3a795ac48a81eb73fb6723328) | 2026-10-21T17:58:02+03:00 | drive: end the AI walk a JOIN recruit brings into the host's drive (manual recruit run desync) | SAV-RECRUIT |
| [5fa67f96](https://github.com/Savas07/KenshiCoop/commit/5fa67f963f4b31d602b449e1f7cadde31fc4b52d) | 2026-10-21T18:01:37+03:00 | Merge branch 'fix/lootrace' into integ/all | SAV-SUPPORT |
| [6c908880](https://github.com/Savas07/KenshiCoop/commit/6c9088806b3c7496aa385360289a7f207dc386e7) | 2026-10-21T18:01:50+03:00 | Merge branch 'fix/recruitrun' into integ/all | SAV-SUPPORT |
| [d5a933aa](https://github.com/Savas07/KenshiCoop/commit/d5a933aaaa78ccdeea7c9052fc4ee32f9f9fda84) | 2026-10-21T18:01:50+03:00 | Merge branch 'fix/redrop' into integ/all | SAV-SUPPORT |
| [19497e70](https://github.com/Savas07/KenshiCoop/commit/19497e70c0a27c6642ae6285d79bf61ed4baefd2) | 2026-10-21T19:27:46+03:00 | Trader lock + shop hand-over (protocol 65, manual T1) | SAV-TRADER |
| [41b09e3f](https://github.com/Savas07/KenshiCoop/commit/41b09e3fbb9206c2290744670574c4cda4d93405) | 2026-10-21T19:27:48+03:00 | Merge branch 'fix/trader' into integ/all | SAV-SUPPORT |
| [3ec2551a](https://github.com/Savas07/KenshiCoop/commit/3ec2551af293eb0e55bbafe7e97854a976d59106) | 2026-10-21T20:24:32+03:00 | Trader hand-over: touched shop lines with absolute counts (protocol 65) | SAV-TRADER |
| [996daa76](https://github.com/Savas07/KenshiCoop/commit/996daa7617e3169fd6fac1397dda0654ef0b2742) | 2026-10-21T20:24:32+03:00 | Merge branch 'fix/trader' into integ/all | SAV-SUPPORT |
| [aa74a2b9](https://github.com/Savas07/KenshiCoop/commit/aa74a2b95041cbc0f1792a1b943784a7c56b3b33) | 2026-10-21T22:14:33+03:00 | v0.58.0: one player at a time per trader + shop stock/money handed over; corpse loot no longer duplicates; join research pays books; save transfer safety; squad names/jobs kept; recruit and inventory desync fixes (protocol 65) | SAV-SUPPORT |
| [4b60185d](https://github.com/Savas07/KenshiCoop/commit/4b60185d3a2651463a13412c132fe87c52d5bdda) | 2026-10-21T22:31:21+03:00 | README: credit the original author and this fork's maintainer | SAV-SUPPORT |
| [58514ea3](https://github.com/Savas07/KenshiCoop/commit/58514ea33538fca78e932abf44bb658d4e915236) | 2026-10-23T14:53:16+03:00 | v0.58.1: backpacks a joining player gains are kept in the host's save; no duplicate bandits after a squad split; fewer NPC/carry desyncs (protocol 65) | SAV-IDENTITY-BAGS |

## Pinned branch tips

| Ref | SHA | Commits ahead of upstream main |
|---|---|---|
| `refs/heads/feature/wanderer-x2-start` | `ac582ae8ee51972361a5672fc83da7a09b8b7382` | 0 |
| `refs/heads/fix/inventory-loss-stabilization` | `6d4dc5efc6baf9f907dc46bf05aa9d9858f78a5a` | 0 |
| `refs/heads/main` | `5a761e19a4184b42315e96ea7e92e3c1403d54db` | 0 |
| `refs/heads/spikes` | `ae44f45224976cd0c66546597b33272b7cb8c776` | 0 |
| `refs/research/DerpyNewb/main` | `5a761e19a4184b42315e96ea7e92e3c1403d54db` | 0 |
| `refs/research/Savas07/main` | `58514ea33538fca78e932abf44bb658d4e915236` | 73 |
| `refs/research/Wo-o-o/dev/stabilize-v0.51` | `c3c1004543e19a4b03afe30405ba798f67832455` | 6 |
| `refs/research/Wo-o-o/main` | `5a761e19a4184b42315e96ea7e92e3c1403d54db` | 0 |
| `refs/research/captainjoshua77-wq/main` | `fc90aff7a7250a6b72ad98f68075ea9e5d55f4ed` | 1 |
| `refs/research/cookingwithsimon/claude/project-thread-tnulw1` | `0c493a94229cda1397c7bffe659503e905bdaa61` | 2 |
| `refs/research/cookingwithsimon/main` | `5a761e19a4184b42315e96ea7e92e3c1403d54db` | 0 |
| `refs/research/ximery4/main` | `58514ea33538fca78e932abf44bb658d4e915236` | 73 |
| `refs/research/ximery4/trader-identity` | `e44ec6545b197bdfecd9526f4dcba8010584f845` | 6 |
| `refs/research/ximery4/upstream/trader-identity` | `13931f29ee3b6f0874ffcec3647416b457a748ed` | 75 |
| `refs/research/ximery4/upstream/xfer-fixture-fab` | `5f89ce8c7e8d03fdfaa25acae53f52e9aeb615ce` | 74 |
| `refs/research/ximery4/xfer-census-source-fab` | `6e62cebc35dde0440046d63c400098809f3c2203` | 75 |
