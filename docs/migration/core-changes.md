# Карта изменений: ядро KenshiCoop и план их совмещения

Снимок публичных git-веток на **1 ноября 2026**. Анализ охватывает nhoral, Savas07, все пять веток ximery4, обе ветки cookingwithsimon и Wo-o-o, captainjoshua77-wq и DerpyNewb. Исходники не исполнялись, DLL не загружались, тестовые скрипты репозиториев не запускались. Проверены git-граф, полные diff, хунки функций, patch-id Wo↔Savas и механическая применимость вложенного captain patch через временный индекс. Возможные дефекты ниже — результаты статического анализа, если явно не сказано иное; они не воспроизведены в игре.

Рекомендуемая база для объединения — [Savas07 `58514ea`](https://github.com/Savas07/KenshiCoop/commit/58514ea33538fca78e932abf44bb658d4e915236), с сохранением всей его истории поверх [nhoral `5a761e19`](https://github.com/nhoral/KenshiCoop/commit/5a761e19a4184b42315e96ea7e92e3c1403d54db). Это **73 коммита, включая 9 merge-коммитов**, 91 изменённый файл, 22 640 вставок и 1 951 удаление. Это существенно более широкая архитектура, чем v0.51: восемь игроков, новая маршрутизация и сообщения, лобби, новые контуры authority и инвентаря. Не стоит воссоздавать её выборочным cherry-pick отдельных исправлений поверх v0.51.

Машинная карта — [changes-core.json](changes-core.json): 31 feature ID, 93 уникальных коммита всего исследованного подмножества, 16 вершин веток; для каждого коммита указаны родители, авторская и коммитерская даты, файлы, контекст функций из diff и feature IDs. Ни один из 73 Savas-коммитов не оставлен вне покрытия. Карта утверждает уникальность только внутри проверенного подмножества; сравнение с Brandon/Tokela/atimal и detached-продолжениями выполняет ведущий агент.

## Git-факты, исключающие повторное внесение изменений

- `ximery4/main` **точно тот же коммит** `58514ea`, что `Savas07/main`; переносить его второй раз нечего.
- `DerpyNewb/main`, `cookingwithsimon/main`, `Wo-o-o/main` **точно тот же коммит** `5a761e19`, что исходный nhoral/main.
- Все три дополнительные ветки nhoral (`feature/wanderer-x2-start`, `fix/inventory-loss-stabilization`, `spikes`) являются предками текущего main; `main..branch` содержит 0 коммитов. Их изменения уже в базе.
- `cookingwithsimon/claude/project-thread-tnulw1`: просмотрены **оба** коммита `768d25b`, `0c493a9` и весь diff. Только `.gitignore` (+ `local/`) и `NOTES.md`; скрытого кода под последним docs-коммитом нет.
- `Wo-o-o/dev/stabilize-v0.51`: **6 коммитов, из них 4 реальных исправления**, затем CI и docs. Последний docs-коммит не отражает объём ветки. Ни один из шести полных patch-id не совпадает с отдельным Savas-коммитом; это не исключает семантическое перекрытие частей.
- `ximery4/xfer-census-source-fab` (`d253e78` + `6e62ceb`) и `ximery4/upstream/xfer-fixture-fab` (`5f89ce8`) имеют **полностью одинаковые деревья** (`git diff --quiet` → 0). Брать одну реализацию один раз.
- Старый `ximery4/trader-identity` (`e44ec65`) имеет **отдельный корень** `83336f8` без общего предка с Savas. Это импорт snapshot v0.58.1 плюс патчи; кроме кода он удаляет `dist/mod-kit` и добавляет gitlink `third_party/KenshiLib_v0.5.0`. Не объединять этот корень merge-ом; две чистые ветки `upstream/*` специально выделяют код для переноса на Savas.
- `captainjoshua77-wq/main` меняет только `ci/fix.patch`. Само хранение патча не применяет его к сборке. Патч имеет внутренний SHA `0f187774...`, а коммит репозитория — `fc90aff...`; не смешивать эти идентификаторы.

## Что уже содержит база Savas: группы реального кода

Все группы `SAV-*` сохраняются базой, а не повторным cherry-pick. Представленные SHA — опорные коммиты группы; полная история всех 73 коммитов и покрытие приведены в конце и JSON. Сценарии, оракулы и последующие коррекции поведения входят в ту же группу; старые WIP и отменённые решения не требуется активировать одновременно с финальным кодом.

1. **SAV-NET8** — `a5b1251`: `MAX_PLAYERS=8`, маски адресатов `PEER_MASK_ALL/peerBit`, host hub/relay, playerId→squad rank, roster presence и selective leave. Основные точки: `Wire.h`, `NetLink::{threadLoop,allocPlayerId}`, `SteamP2P.cpp`, `Inbound.h`, `OwnRanks::resolveOwnRanksForPlayer`, `Plugin::processNetEvents`. Это зависимость почти всех последующих групп.
2. **SAV-LOBBY** — `a5b1251`, `6df1084`, `d7a3cc0`, `de1d5c2`, `aaea80a`: Steam invites/rich presence, ожидание ready world, main-menu MULTIPLAYER, вход без локальных сохранений. `SteamInvite::{onLobbyEnter,onLobbyCreated,tick}`, новый `EngineLobby.cpp`, `EngineUi.cpp`, `Plugin::{coopPanelDrive,titleUpdate_hook,noteWorldLive}`. Не заменять позднее целиком UI-файлами более старой ветки.
3. **SAV-CHARGEN** — `2e1f5ce` + завершающий `433f327`, затем `6fb51c0`, `de1d5c2`, `de30237`: родной editor гостя, `.bod2` appearance/name/race в `CharGenHeader`, разрешение только fresh New Game. `EngineCharGen.cpp`, `Plugin::tickCharGen`, `LoadGoPacket::freshStart`, `NetLink` dispatch. Один WIP `2e1f5ce` сам по себе прямо не собирается; сохранять завершённый путь.
4. **SAV-ROSTER** — `be1a051`, `3a567b9`, `29090d9`: стабильные Steam identities, `kenshicoop_players.txt` в save, имена вкладок и передача отсутствующего игрока заместителю. `RosterPacket`, `NetLink::allocPlayerId`, `EngineEntity.cpp`, `Plugin::applyHostRanks`, `ReplicatorPublish::publishOwned`. Совмещение с альтернативным владением отрядами требует политики, а не суммы присваиваний.
5. **SAV-SAVE** — `88ec951`, `6fb51c0`, `de1d5c2`, `0c90ef8`: Unicode Win32 APIs; bootstrap save не перезаписывает существующий; BEGIN/chunk/DONE ordering; unsafe save name/path refusal; наличие `quick.save`; CRC/seen-file проверка; `__incoming`→final через `__old` с восстановлением stranded save. `SaveXfer::{onSaveBegin,onSaveFile,onSaveDone,recoverStrandedSave,tickSend}`, `Plugin::pumpSaveReceive`. Это безопасность commit, **не** полный lifecycle cancel; WO-ABORT дополняет её.
6. **SAV-DRIVE** — `433f327`, `ec89764`, `86b95e0`, `4adca37`, `fa71c22`: movement samples, smoother glide с анимацией, get-up и stale seat/work pose release, carrier carve-out, Steam crash guard. `ReplicatorDrive::{applyTargets,pruneDriveGrace}`, `Interp.h`, `ReplicatorUtil.h`, `EngineCharState.cpp`, `SteamP2P.cpp`. Последовательные test-коммиты являются историей доведения одного поведения.
7. **SAV-DEVTOOLS** — `4f185bc`, `fff395a`: overlay, packet recorder, watchdog/desync dump, telemetry UDP, dashboard и replay/inspection tools, scripted `move` repro. `src/plugin/devtools/*`, `ReplicatorDev.cpp`, `tools/devkit/{dashboard,replay}`, `Config`. Сохранять как опциональный комплект: master switch по умолчанию OFF, replay — отдельный tool, не игровой сетевой сервис.
8. **SAV-VENDOR** — `3198745`, `19e2f62`, `ebbc6f0`, `e6e1ddd`, `7c6e19c`: shared shop counters, vendor deltas/cash host-authoritative, inventory buy/place-once. `VendorDeltaPacket`, `VendorCashPacket`, `EngineInventory::{vendorBuyInto,placeOnce}`, `ReplicatorItems::{publishInventories,detectAndPublishVendorDeltas,applyVendorDeltas,applyVendorCash}`. Counter stock и trader-window агрегат — разные пути, следующий SAV-TRADER не заменяет этот контур.
9. **SAV-PRODUCTION** — `e5ad2ef` + `f80779e`: только host симулирует production/crafting output; re-pose lapsed machine copies. `EngineInternal::{operateProd_hook,operateCraft_hook,opHostSimulates,installMachineOperateHooks}`, `ReplicatorChannels::applyProd`, `ReplicatorDrive::applyRest`, machine template probes. Не распространять guard без разбора на любую Building::operate: финальный код сужает его до production/crafting.
10. **SAV-RECRUIT** — `0b8f63f`, `3e9bd90`, `5c5f388`: recruit из host-driven NPC получает правильного владельца, wire-space subject, owner pin, запрет публикации squad body как world NPC, сброс старого AI walk. `recruit_hook`, `publishRecruits`, `releaseRecruitDrive`, `publishOwned`, `applyTargets`. Зависит от общей identity схемы и непременно проверяется вместе с NPC duplication.
11. **SAV-COMBAT** — `337004a`, `5d84583`, `65b81d5`/`d2509ed`, финальный `ef40acd`: target resolve через body mapping, per-owner-target reissue budget; engaged squad copies не получают walk order/sustained snap в бою. `EngineSpawnCombat::{applyCombat,forceAttack}`, `ReplicatorDrive::applyTargets`, `ReplicatorUtil.h`. Два противоположных исторических re-order коммита не являются двумя параллельными фичами; основа — финальное поведение `ef40acd` и последующие fixes.
12. **SAV-WORLD** — `d82a496`: структурный baseline всех scan-only save-native ground items, unresolved/dormant own drop tracking, native pickup notice `PKT_NATIVE_TAKEN` вместо второй копии. `ReplicatorItems::{publishWorldItems,applyNativeTakes,rememberDrop,takeRecentDrop}`, `Wire.h`, `Inbound`, `NetLink`. Не заменять более простой temporal baseline из старых веток.
13. **SAV-CORPSE** — `0c3def3`: census dead non-squad inventories, `INV_FLAG_CORPSE`, author body stream key и proxy resolve. `EngineWorld` corpse enumeration, `ReplicatorItems::{publishInventories,ingestInv}`, config. Нужен общий inventory truth, а не локальная копия трупа.
14. **SAV-RESEARCH** — `a6c97be`: grow-only union исследованных tech, join research bench расходники идут host через vendor delta infrastructure. `publishResearch/applyResearch`, `detectAndPublishVendorDeltas`, `EngineInventory::researchPickSubject`. Сохранять и объединение tech, и стоимость: одна половина не заменяет другую.
15. **SAV-JOINSTATE** — `7697515`: squad name/faction name/permanent jobs join переживают host-authored save; `JoinStatePacket/JS_*`, seq per sender/kind/key и перевод job target building keys. `ReplicatorChannels`, `EngineCharState`, `EngineInternal`, `Replicator::forgetPeerChannels`. Имена — last-writer state с host policy, jobs — join→host; при совмещении политик это должно быть явно сохранено.
16. **SAV-XFER-RACE** — `3244070`: короткий author source означает, что вещь уже забрана, а не разрешение создать вторую; optimistic loser возвращается через rejected/partial ACK/reconcile. `ReplicatorItems::{applyTransfers,applyXferAcks}`, `corpse_loot_race` scenario. XIM-XFER меняет ровно этот инвариант для части source types, поэтому порядок переноса не заменяет проверку сохранности вещей.
17. **SAV-GEAR** — `4bd312f`: actual gear object identity после drop, фиксация последней вещи из ставшего пустым инвентаря. `EngineInventory::dropItemFromInventory`, `ReplicatorItems::detectAndPublishWeaponDrops`; сохранить W2 conservation transport.
18. **SAV-TRADER** — `19497e7`, `3ec2551`: host lock одного trader, touched shop lines с абсолютным count + absolute money при hand-over. `TradeHeader`, `ReplicatorItems::{tickTradeLock,sendTradeOp,applyTradeTable,applyShop}`, `EngineWorld`, `EngineInventory::closeTradeWindow`. Raw trader hand в этом коде ошибочно предполагается save-stable между машинами; XIM-TRADER вводит другой resolve.
19. **SAV-IDENTITY-BAGS** — большой завершающий `58514ea`: (а) приобретённый backpack после `WD_REHOME_MAX_MS+4000` может создаться из snapshot с nested contents, сохраняя grace реального W2 relocation; (б) `indexProxies/wireKeyOf/publishKeyFor/pruneLastHands/xlateSubjects`, стабильное имя NPC после squad split; (в) `identityKeyOf`, wire/body resolve для medical, treatment, combat hit и carried subject; (г) cell-claim spatial hysteresis; (д) не freeze-driven proxy и не park-carried body; (е) auto-pause dialogue в первые 1000ms не становится глобальным speed vote. Главные файлы: `Replicator{Publish,Authority,Channels,Items,Spawn,Drive}.cpp`, `EngineInventory.cpp`. Эта группа многотемная и её нельзя полностью заменить одним похожим fix из другого форка.
20. **SAV-SUPPORT** — сценарии и оракулы всех названных контуров, localized machine template probing, debug snapshots, UI/version/docs/credit, `.mod` генератор 2..8-player starts, release artifacts. `scripts/scenarios.psd1`, `oracles/{Combat,Inventory,World,Npc,Motion}.ps1`, `Scenario{Combat,Inventory,WorldItems,Npc,Buildings,Probes}.cpp`, `EngineProbe`, `prototest`, `tools/MultiplayerStartGen`. Сохранять исходники/проверки; `.dll`/`.mod` из разных dist каталогов не смешивать: итоговый комплект нужно заново собрать/сгенерировать из одного commit.

## Wo-o-o: четыре исправления и что именно портировать

### WO-SNAPSHOT — `b0773ae4`

[Коммит](https://github.com/Wo-o-o/KenshiCooop/commit/b0773ae4) меняет `EngineInventory::readInvItems` и `applyContainerContents`.

- Если `g_getSectionsFn` отсутствует/не дал sections, snapshot помечается incomplete, а не полным пустым списком. После заполнения output продолжает искать пропущенные nested entries, чтобы корректно выставить truncation.
- Перед мутацией проверяется parent graph: диапазон, self-parent и вложенный parent запрещены.
- Полный snapshot с parent bag без children означает пустой bag. Truncated snapshot не даёт права очищать пропущенных детей. Несколько одинаковых local bags или truncated parent recapture дают `NESTED-EMPTY-DEFER`, а не очистку случайного экземпляра.
- В Savas `applyContainerContents` уже имеет новые `allowContainerCreate/outContainerShort` и grace. Сохранить эти параметры и bag creation, внедрить проверки и nested-empty path **ручным портом**, без замены файла старой версией.

### WO-LATCH — `975d2dbe`

[Коммит](https://github.com/Wo-o-o/KenshiCooop/commit/975d2dbe) добавляет header-only `compactInventorySnapshot` и меняет `ReplicatorItems::{applyInventories,applyXferAcks}`.

- Latch tombstones удаляются после всех правок; surviving parentIdx перенумеровываются. Дети удалённого parent также убираются. Невалидный/переполненный граф откладывает reconcile.
- Catch-up требует полных remote и local snapshots. Добавленный синтетический backpack parent делает snapshot additive-only, потому что у него нет достоверного списка children.
- ACCEPT подтверждает mutation, а не получение свежего snapshot. Поэтому accepted latch сохраняется до catch-up/deadline; rejected/partial освобождаются для rollback.
- Сохранить Savas shop/research/corpse rebase branches и `bagShortMs`. Проверить totals только соответствующего уровня вложенности: патч фильтрует desired top-level, а local sum требует отдельного просмотра при порте. Это интеграционная проверка, не установленный дефект.

### WO-WORLD — `3deedd1f`

[Коммит](https://github.com/Wo-o-o/KenshiCooop/commit/3deedd1f) пересекается с SAV-WORLD, но не равен ему.

- Структурный scan-only native baseline и `pendingDrops_` **семантически уже есть** у Savas (`d82a496`), причём у Savas есть дополнительные dormant drops и `NativeTakenPacket`. Их сохранять, не вносить второй несовместимый baseline.
- Уникальные относительно проверенного Savas хунки: `consumedWorldItems_` tombstone `(ownerId,netId)` после локального pickup, блокирующий resurrection от in-flight item snapshot; очистка на reset; ранний ownerClass==peer фильтр **до** unresolved drop bookkeeping; `captureWorldItems` ownership filter для non-player faction с исключением sentinel stringID `nofac`.
- У Savas `consumedWorldItems_` и этот faction filter отсутствуют. Его `worldTrack.claimed` защищает автора от повторного CLAIM, но не закрывает тот же receive-side replay window, потому это не точный дубль.
- Перенести только недостающие инварианты, сохранив 8-player scoping, `NativeTaken`, dormant native/own item distinction, claimed state и invalid pointer protections. Tombstone нужен до конца **world epoch**, а не глобально между новыми играми; проверить sender id/netId reuse после reconnect.

### WO-ABORT — `1125031c`

[Коммит](https://github.com/Wo-o-o/KenshiCooop/commit/1125031c) добавляет `sendAbortAndNotify`, `abortReceive`, `SessionQ::clearQueue`, `Inbound::discardSaveLoadState`, вызовы из UI reset/leave, подробные Win32 I/O ошибки.

- Sender open/read failure посылает `SaveDoneHeader` с fileCount=0. Receiver завершает failed transfer без commit; бесконечный progress прекращается. У Savas `onSaveDone` уже безопасно отвергает empty DONE (`0c90ef8`), но sender failure всё ещё вызывает только local `sendAbort`.
- `abortReceive` закрывает file handle, снимает active, оставляет staging для диагностики, двигает commit result/seq. `abortAll` включает receiver. У Savas ни этого helper, ни вызовов на disconnect нет.
- Actual disconnect очищает BEGIN/FILE/DONE/ACK/LOAD GO/REQ/NACK queues; world reload, сохраняющий connection handshake, не должен чистить их тем же способом.
- **Нельзя буквально брать двухигроковый `if (!leaves.empty()) abortReceive+discard all`:** Savas host получает selective leave одного join. Сохранить pending работу оставшихся игроков. Sender failure должен использовать `g_sendMask`/target peer identity, не безусловный broadcast. Идеальная единица cancel — `(sessionEpoch, peerId, transferId)`; на join при потере host cancel всей receive session допустим.
- Сочетать safe commit Savas и lifecycle cancel Wo; это дополнительные, а не альтернативные меры.

### WO-CI — `15c25077`, docs `c3c10045`

CI добавляет portable mocks и meaningful cases: `InventoryRegression`, `SnapshotRegression`, `WorldRegression`, `SaveAbortRegression`, sanitizer runner, существующий prototest, `check_re_integration.py`, build_snapshot_tests и отдельный genuine Windows v100 integration gate. Workflow Windows job выполняется только явным dispatch на prepared runner; portable mocks не подтверждают ABI Kenshi.

Портировать тестовые сценарии вместе с функциями, адаптировать signatures Savas и multi-peer cancellation; не объявлять устаревшую mock ABI проверкой Release DLL. Документацию technical audit/RE_KENSHI integration сохранить с атрибуцией, переименовать repository identity под итоговый проект. В этой сессии эти тесты не запускались.

## Ximery4: две чистые патчветки и недостающая диагностика старой

### XIM-TRADER — `88f0dd9` → `13931f2`

[Чистая ветка](https://github.com/ximery4/KenshiCoop/tree/upstream/trader-identity), основание точно `58514ea`. `88f0dd9` меняет `Wire.h`, `Replicator.h`, `ReplicatorItems.cpp`, `prototest`; `13931f2` выделяет несовместимый protocol 165.

`TradeHeader` получает `TRADE_FLAG_DESC`, `tSid[48]`, `tFac[48]`, position; raw sender hand переводится `traderLocalKey` в local hand. Сначала cached pair, затем raw-hand template+position match, затем `adoptCandidateNear`; radius принятия 20u. Не найденный trader получает foreign key с high bit и sender namespace. Полученный join RELEASE c stock паркуется до загрузки trader: retry раз в секунду, retention 10 минут, максимум 64 записи. `tickTradeLock` разделён на `handleTradeOp` и `tickTradeLocal`. `KENSHICOOP_TRADE_SCRAMBLE` под HARNESS заставляет проверять cross-machine path и при двух локальных клиентах.

**Перенос:** можно взять два коммита как исходную единицу, но до выпуска обязательна доработка lifecycle/identity:

- Новые `tradePeerKey_`, `tradeDesc_`, `tradePark_`, `tradeMissLog_` **нигде не очищаются** при resetSession. Их usages проверены git grep; `ReplicatorCore.cpp` патч не меняет. Старый parked stock table может пережить смену мира и найти одноимённый trader в новом. Это статически обнаруженный риск, не воспроизведённый баг. Добавить epoch, clear на world/session reset, selective erase departed peer.
- Cache hit проверяет только resolve local body; требуются descriptor/epoch validity и тест recycled hands. Raw-hand fast path сравнивает sid и distance, но не faction; nearest candidate path не доказывает уникальность среди нескольких одинаковых торговцев. Нужны reject-ambiguous semantics и тест два одинаковых trader в пределах radius, пересечение этажей/Y, перемещение trader и unload/reload.
- Равный протокол на двух разошедшихся ветках не означает совместимость. При совмещении всех форков выделить один итоговый wire version/schema, обновить размеры/dispatch/round-trip tests и reject mismatches. Не сохранять «165 гарантирует совместимость» без единой спецификации.

### XIM-XFER — `5f89ce8` либо `d253e78`+`6e62ceb`

[Чистый коммит](https://github.com/ximery4/KenshiCoop/commit/5f89ce8c7e8d03fdfaa25acae53f52e9aeb615ce) меняет только `ReplicatorItems::applyTransfers` и `[xfer] APPLY` diagnostic fields. Дерево равно двухкоммитной `xfer-census-source-fab` целиком.

Новый `srcCensusOnly` разрешает fabricate source shortfall даже при `sAuth==localId`, если `moved==0`, `sk.t==0`, source есть в `censusContainers_`, отсутствует в `censusCorpses_`, `ownHands_`, `ownedContainers_`. `KENSHICOOP_XFER_CENSUS_FAB` default ON, `=0` возвращает старое правило. Начальный широкий патч включал corpse и был сужен последним коммитом после описанного автором corpse-loot dupe; нужно переносить **финальную**, а не первую форму.

**План:** сохранить исправление как экспериментальный перенос, но не автоматически активировать в объединённой сборке. Отсутствие локальной вещи не доказывает, что вещь не была уже забрана: type0 включает shared chest/box, поэтому статически остаётся возможный chest race, обходящий SAV-XFER-RACE. Сам этот дефект не воспроизводился нами. Более устойчивое условие — доказанная production-source provenance/author output transaction и idempotent token, а не один `moved==0`. Обязательные проверки: одновременный loot emptying chest, corpse race, campfire/node yield first take, повтор ACK/transferId, partial quantity, disconnect/rejoin. Сохранить авторский feature flag, до подтверждения default OFF или diagnostics-only.

### XIM-DIAG — `9eedc08`

Старый orphan `trader-identity` дополнительно имеет `lastAddItemsFailReason`/`g_addFailReason` в `EngineInventory`, объявление в Engine.h и `fabWhy` в applyTransfers. Причины: bad args, unresolved container hand, no inventory, item/template creation failure, skipped. Этой диагностики **нет** в двух чистых upstream/* ветках, поэтому механическое взятие только их упустит уникальное изменение. Перенести вручную маленьким отдельным диагностическим патчем с сохранением logging contract; global static допустим только при гарантии main-thread вызовов. Остальные старые trader patches (pairing/scramble и final fixture restriction) уже отражены чистыми ветками; orphan snapshot packaging/gitlink не является новой игровой фичей.

## Captain: два независимых смысла вложенного патча

[`fc90aff`](https://github.com/captainjoshua77-wq/KenshiCoop/commit/fc90aff7a7250a6b72ad98f68075ea9e5d55f4ed) хранит patch на 5 исходных файлов, 64 insertions/3 deletions. `git apply --cached --check` на временном индексе upstream main проходит; на Savas не проходит контекст `Replicator.h`, `ReplicatorAuthority.cpp`, `ReplicatorCore.cpp`. Это проверка применимости текста, не сборки.

**CAP-REKEY.** Добавляет session-lifetime reverse map `wireOfLocal_` LOCAL→WIRE; заполняет при обнаружении rekey в applyTargets, wide census consult перед cull, clear на resetSession. В Savas `identityKeyOf`/`canonicalOf_` уже закрывает часть той же проблемы, но `pruneDriveGrace` удаляет `canonicalOf_` через **30 секунд**. Captain намеренно сохраняет mapping после ухода stream, поэтому нельзя без проверки пометить патч целиком «уже есть». Портировать долгоживущий инвариант в общую serial-checked identity service: учесть Savas proxyRev_/lastHand_, session/world epoch, re-rekey, object destruction, pointer/hand reuse и peer leave. Тест: rekey body в бою → уйти за interest/stream минимум на 30 секунд → wide census → вернуться; отсутствие blink/duplicate/cull.

**CAP-DOOR.** `publishDoors`: minted standalone proxy door публикуется через `mintByLocal_` в placer key space. `applyDoors`: incoming placer key разрешается `peerBuilds_`→localHand, local `doorRows_` получает echo baseline; read/write используют local hand. Это для building, который **сам является дверью**, без parent building, а не обычного build-door protocol28. Эти хунки в Savas отсутствуют; перенести отдельно, с тестами host-placer и join-placer, open/lock, no echo, proxy remove/unload. Wire layout не меняется. Даже если Channels хунки механически применяются, это не основание пропустить semantic testing.

## Порядок совмещения с сохранением всех уникальных изменений

1. **Contracts и epochs.** Зафиксировать Savas585 и SHA всех источников; единые player/source/body/fixture key types, world/session/transfer epochs; единый wire schema и version. Сохранить тесты packet layouts, enums и dispatch. Новые поля/пакеты других форков нельзя складывать по номеру version.
2. **Save safety.** Сохранить SAV-SAVE, добавить WO-ABORT с selective peer cancel и targetMask; адаптировать portable regressions и реальные save-transfer сценарии. Не возвращать старые narrow APIs или broadcast-abort двухигроковой версии.
3. **Identity.** Общая serial-validated mapping с lifespan; CAP-REKEY long-tail + CAP-DOOR; XIM-TRADER descriptors, foreign namespace и epoch-scoped parked tables; сохранить все SAV-IDENTITY-BAGS ветви для medical/carry/authority. Только после этого решать поведенческие конфликты с другими форками.
4. **Conserved item transactions.** WO-SNAPSHOT → WO-LATCH → недостающие WO-WORLD хунки, сохраняя Savas bag creation grace, W2 relocation, corpse source truth и native pickup. XIM-XFER — отдельная экспериментальная policy поверх тех же инвариантов; XIM-DIAG сохраняется независимо.
5. **Intents/combat.** Сохранить конечную SAV-COMBAT/SAV-DRIVE логику, не активировать исторические revert-пары. Подмешивать из других форков по owner/body authority, а не целым старым ReplicatorDrive.
6. **Multi-peer.** Все новые queues/maps/cancels проверить для host + минимум двух joins, отдельный leave/rejoin не стирает остальных; peer id reuse не пересаживает stale tables/tombstones. Savas net8 уже база, но каждый порт должен сохранять её контракт.
7. **Gameplay policies.** Сохранить opt-in devtools и флаги; по отдельности оформить альтернативные production/loot/clock/host-role политики остальных форков. «Все изменения» означает сохранение всех осмысленных реализаций/инструментов с явными вариантами, а не одновременное применение противоположных правил.
8. **UI/transports и release.** Сохранить Savas lobby/roster/chargen и дополнительные UI/transports через adapters. Cooking NOTES/Wo identity docs — provenance, не runtime; rebuild DLL/.mod и пакет из единой вершины, никакого выбора бинарника из случайного dist каталога.

Минимальные regression gates перед готовностью будущей интеграции: genuine v100 Harness+Release build; prototest packet/queue contracts; inventory mock tests; реальные scenario/oracle группы из обеих линий; два физических ПК для различающихся hands/Steam trader path; минимум 3-player leave/rejoin/save; сохранение итогового мира и повторная загрузка с bags/jobs/roster. Эти проверки пока **планируются**, не отмечены выполненными. Исследование и механические git checks не доказывают игровую стабильность.

## Полная таблица feature IDs

| ID | Действие | Опорные SHA | Зависимости |
|---|---|---|---|
| `SAV-NET8` | `included_base` | a5b12518 | — |
| `SAV-LOBBY` | `included_base` | a5b12518, 6df1084e, d7a3cc07, de1d5c22, aaea80a6 | SAV-NET8 |
| `SAV-CHARGEN` | `included_base` | 2e1f5ce4, 433f3275, 6fb51c09, de1d5c22, de302373 | SAV-NET8, SAV-SAVE |
| `SAV-ROSTER` | `included_base` | be1a0519, 3a567b92, 29090d90 | SAV-NET8 |
| `SAV-SAVE` | `included_base` | 88ec9518, 6fb51c09, de1d5c22, 0c90ef83 | — |
| `SAV-DRIVE` | `included_base` | 433f3275, ec897642, 86b95e08, 4adca37c, fa71c222 | SAV-NET8 |
| `SAV-DEVTOOLS` | `included_base_opt_in` | 4f185bcc, fff395ae | — |
| `SAV-VENDOR` | `included_base` | 3198745f, 19e2f625, ebbc6f0e, e6e1ddde, 7c6e19c3 | SAV-NET8 |
| `SAV-PRODUCTION` | `included_base` | e5ad2ef0, f80779ed | SAV-NET8 |
| `SAV-RECRUIT` | `included_base` | 0b8f63f8, 3e9bd90b, 5c5f3886 | SAV-NET8 |
| `SAV-COMBAT` | `included_base` | 337004a2, 5d845833, 65b81d53, d2509ed3, ef40acd5 | SAV-DRIVE |
| `SAV-WORLD` | `included_base` | d82a496d | SAV-NET8 |
| `SAV-CORPSE` | `included_base` | 0c3def37 | SAV-VENDOR |
| `SAV-RESEARCH` | `included_base` | a6c97be0 | SAV-VENDOR |
| `SAV-JOINSTATE` | `included_base` | 76975155 | SAV-NET8, SAV-ROSTER |
| `SAV-XFER-RACE` | `included_base` | 32440706 | SAV-CORPSE |
| `SAV-GEAR` | `included_base` | 4bd312f8 | — |
| `SAV-TRADER` | `included_base` | 19497e70, 3ec2551a | SAV-VENDOR |
| `SAV-IDENTITY-BAGS` | `included_base` | 58514ea3 | SAV-RECRUIT, SAV-DRIVE, SAV-COMBAT |
| `SAV-SUPPORT` | `included_base_rebuild_artifacts` | 1c147364, c939bcce, f76021a4, 79e4645f, 73518a01, 74049b22, 01f9d227, eecb4e2e, 842160eb, 876ec41b, 58472712, 5cac89bf, 3b1ad984, 77026f44, 47e99af0, ded25a75, 50fcd027, aa74a2b9, 4b60185d, 5e781654 | — |
| `WO-SNAPSHOT` | `manual_port` | b0773ae4 | SAV-IDENTITY-BAGS |
| `WO-LATCH` | `manual_port` | 975d2dbe | WO-SNAPSHOT |
| `WO-WORLD` | `manual_port_unique_hunks` | 3deedd1f | SAV-WORLD |
| `WO-ABORT` | `manual_port` | 1125031c | SAV-SAVE, SAV-NET8 |
| `WO-CI` | `port_and_adapt_tests` | 15c25077, c3c10045 | WO-SNAPSHOT, WO-LATCH, WO-WORLD, WO-ABORT |
| `XIM-TRADER` | `cherry_pick_then_lifecycle_fix` | 88f0dd94, 13931f29 | SAV-TRADER |
| `XIM-XFER` | `take_once_experimental_manual_port` | 5f89ce8c, d253e787, 6e62cebc | SAV-XFER-RACE, SAV-PRODUCTION |
| `XIM-DIAG` | `manual_port_optional_diagnostics` | 9eedc089 | XIM-XFER |
| `CAP-REKEY` | `manual_port_not_already_fully_covered` | fc90aff7 | SAV-IDENTITY-BAGS |
| `CAP-DOOR` | `manual_port` | fc90aff7 | — |
| `COOK-NOTES` | `docs_optional_preserve_provenance` | 768d25be, 0c493a94 | — |

## Покрытие всех 73 коммитов Savas

Указана топологическая история от общего upstream main до Savas585. Merge-коммиты и сопровождающие тесты/документы имеют SAV-SUPPORT; их конкретные изменённые файлы и diff context доступны в JSON.

| № | SHA | Тема | Feature IDs |
|---|---|---|---|
| 1 | [a5b12518](https://github.com/Savas07/KenshiCoop/commit/a5b1251891bb7022a33cc09652aad88daacc7863) | coop: up to 8 players + one-click Steam invites (protocol 56) | SAV-NET8, SAV-LOBBY |
| 2 | [6df1084e](https://github.com/Savas07/KenshiCoop/commit/6df1084e8f99bdc28b3379f84b0cb43d759681f3) | coop: friends wait in the lobby until the host's world is live, then connect | SAV-LOBBY |
| 3 | [d7a3cc07](https://github.com/Savas07/KenshiCoop/commit/d7a3cc07c6362a79e73a999d4d13c6965bd53b51) | coop: a join with no local saves can enter the host's world | SAV-LOBBY |
| 4 | [1c147364](https://github.com/Savas07/KenshiCoop/commit/1c147364c5886dd0ef8cd99d0d6efe1608e7b270) | coop: joining players log as [JOIN]; kit README says where the log is | SAV-SUPPORT |
| 5 | [88ec9518](https://github.com/Savas07/KenshiCoop/commit/88ec95183e74c9d6977d660def6e87f318cca48d) | savexfer: Unicode file APIs so translated saves transfer | SAV-SAVE |
| 6 | [2e1f5ce4](https://github.com/Savas07/KenshiCoop/commit/2e1f5ce421272a899950c72936a19715110394c8) | WIP v0.55: version constant, MoveSync partial (does not build yet) | SAV-CHARGEN |
| 7 | [433f3275](https://github.com/Savas07/KenshiCoop/commit/433f3275ed7c46a2a4175cd129b4182ffdb51d94) | v0.55: guest character creation, smoother remote movement, Steam crash guard | SAV-CHARGEN, SAV-DRIVE |
| 8 | [6fb51c09](https://github.com/Savas07/KenshiCoop/commit/6fb51c092b584bc7cbd34718fe50cfac9f162e35) | v0.55.1: character creation only on worlds the host started with New Game; save-transfer DONE/chunk race fix | SAV-CHARGEN, SAV-SAVE |
| 9 | [de1d5c22](https://github.com/Savas07/KenshiCoop/commit/de1d5c22dbf247d52d79410a2145595191c006e4) | v0.56: main-menu MULTIPLAYER lobby; new-game connect push no longer overwrites a save | SAV-LOBBY, SAV-CHARGEN, SAV-SAVE |
| 10 | [de302373](https://github.com/Savas07/KenshiCoop/commit/de3023734003a35c62d3a74db750481e128a70c7) | v0.56.1: track New Game freshness on every world-live edge, whatever the role | SAV-CHARGEN |
| 11 | [aaea80a6](https://github.com/Savas07/KenshiCoop/commit/aaea80a6ec3f12bdb408d5ba1c6b7b05015e3474) | v0.56.2: Steam rich presence for joining + invite/join diagnostics | SAV-LOBBY |
| 12 | [be1a0519](https://github.com/Savas07/KenshiCoop/commit/be1a05193b8578b683a8dc704f341dafd31e6618) | v0.57: squad tabs named after their players; who-had-which-squad remembered in the save | SAV-ROSTER |
| 13 | [3a567b92](https://github.com/Savas07/KenshiCoop/commit/3a567b9259017bddbe7c11a4fd6f65055e7caa6a) | v0.57.1: a substitute player takes the absent player's squad | SAV-ROSTER |
| 14 | [29090d90](https://github.com/Savas07/KenshiCoop/commit/29090d908778ad775068dd3d210c8cd3a908af10) | v0.57.2: squad tab bar shows the player names at once; status texts point to MULTIPLAYER | SAV-ROSTER |
| 15 | [5e781654](https://github.com/Savas07/KenshiCoop/commit/5e7816542e590e89301bdf1a0e6a993b0b9849d5) | v0.57.3: hide the top-left status banner (KENSHICOOP_BANNER=1 shows it for debugging) | SAV-SUPPORT |
| 16 | [ec897642](https://github.com/Savas07/KenshiCoop/commit/ec89764294f50e1ded464c015acb828bd2ce783c) | v0.57.4-test (local, NOT released): squad glide + get-up release | SAV-DRIVE |
| 17 | [86b95e08](https://github.com/Savas07/KenshiCoop/commit/86b95e08f866277ec3e4e0740956b6dee384b9f8) | v0.57.4-test2 (local, NOT released): glide keeps the model and the gait | SAV-DRIVE |
| 18 | [4adca37c](https://github.com/Savas07/KenshiCoop/commit/4adca37c471e1a3e0382393b52818d596b773567) | v0.57.4-test3 (local, NOT released): never glide a carrier | SAV-DRIVE |
| 19 | [4f185bcc](https://github.com/Savas07/KenshiCoop/commit/4f185bcc20e877b06988127ff042658f551422d3) | dev/tools (local, NOT released): developer tools for co-op sync debugging | SAV-DEVTOOLS |
| 20 | [fff395ae](https://github.com/Savas07/KenshiCoop/commit/fff395aeaa49b740c10fc6df6d4e8491ad5f03a3) | dev/tools: move <hand> <dx> <dz> command for scripted repros | SAV-DEVTOOLS |
| 21 | [c939bcce](https://github.com/Savas07/KenshiCoop/commit/c939bcce42aa0d77b9341c032efe216b8156b3de) | v0.57.5: no more teleports/frozen models/animationless sliding for friends' characters; carried bodies stay visible; dev tools (off by default) | SAV-SUPPORT |
| 22 | [79e4645f](https://github.com/Savas07/KenshiCoop/commit/79e4645f1198a2e6275b46484e7d7a1c21a8511a) | recruit_join: scenario + Test-RecruitJoin oracle (join recruits a host-driven NPC; host-recruit control arm) | SAV-SUPPORT |
| 23 | [0b8f63f8](https://github.com/Savas07/KenshiCoop/commit/0b8f63f8de94288ed12cf5e13738ab6007b1c416) | recruit: a join recruit of a host-driven NPC is the join's on both games | SAV-RECRUIT |
| 24 | [01f9d227](https://github.com/Savas07/KenshiCoop/commit/01f9d22711386f4a8911a09059a5539313040150) | docs: pitfall 19 - name an identity edge's subject in the receiver's key space (recruit_join) | SAV-SUPPORT |
| 25 | [3e9bd90b](https://github.com/Savas07/KenshiCoop/commit/3e9bd90b66188bd99ac08e8d873389f2b9bab86a) | publishOwned: never stream a pinned squad hand as a world NPC (a just-inserted peer recruit passed captureNpcs' playerCharacters filter for one tick; the recruiter REQ'd its fresh hand and minted a phantom twin into its own squad - rj_after2/rj_after_wan) | SAV-RECRUIT |
| 26 | [5cac89bf](https://github.com/Savas07/KenshiCoop/commit/5cac89bf9ebbdda81add81a9fe1168177744fed6) | recruit_seated: seated arms of recruit_join (both sides recruit a posed bar NPC); oracle gates posed subject + recruiter phantom twin | SAV-SUPPORT |
| 27 | [fa71c222](https://github.com/Savas07/KenshiCoop/commit/fa71c2220ace43c7b6c5ba8021ad76caa6c17be5) | drive: end a squad copy's stale seat/work pose once its owner walks with no task ([heal] POSE) - the peer's copy of a seated recruit sat frozen or crawled | SAV-DRIVE |
| 28 | [ded25a75](https://github.com/Savas07/KenshiCoop/commit/ded25a75406ea5a4549d1c8357efabfe42b91b89) | docs: pitfall 19 - receiver-side NPC-SKIP and seat-pose heal follow-ups | SAV-SUPPORT |
| 29 | [ee6da702](https://github.com/Savas07/KenshiCoop/commit/ee6da70291117e2495ef7163046bd6803bd52739) | Merge branch 'fix/recruit' into integ/all | SAV-SUPPORT |
| 30 | [3198745f](https://github.com/Savas07/KenshiCoop/commit/3198745f76e67edf6a64f9dbe68eefcd6e636075) | vendor stock: shared shop counters (protocol 60) + vendor_stock scenario | SAV-VENDOR |
| 31 | [eecb4e2e](https://github.com/Savas07/KenshiCoop/commit/eecb4e2e6bf8165f5874a663dff0580fc0ab93d9) | Test-VendorStock: take the earliest purchase time without Measure-Object -Property on hashtables | SAV-SUPPORT |
| 32 | [58472712](https://github.com/Savas07/KenshiCoop/commit/584727121a5e547fb0a9fce2d75e0b43695913e5) | vendor_stock: sell leg (host + join sell one stack back) and merchant-cash gate | SAV-SUPPORT |
| 33 | [19e2f625](https://github.com/Savas07/KenshiCoop/commit/19e2f62546cea77308515c6d557f9d4860e5f7f6) | vendor stock: host-authoritative merchant cash (protocol 60) | SAV-VENDOR |
| 34 | [ebbc6f0e](https://github.com/Savas07/KenshiCoop/commit/ebbc6f0eaee3d818d9e6c4292c4fa7b8ed3a6337) | vendorBuyInto: place a traded stack exactly once (loose section first) | SAV-VENDOR |
| 35 | [e6e1ddde](https://github.com/Savas07/KenshiCoop/commit/e6e1dddebfd2098722a21aaff4d2cffb4bab1f39) | vendor_stock: place a traded stack exactly once; sell through the trade window's shop view | SAV-VENDOR |
| 36 | [7c6e19c3](https://github.com/Savas07/KenshiCoop/commit/7c6e19c387035cfb8620b797601f86fdd67819a3) | vendor_stock: drop the unexercised shop-view sale path; a sale is buyItem + place-once | SAV-VENDOR |
| 37 | [03d17404](https://github.com/Savas07/KenshiCoop/commit/03d17404d9ab15b8776f53e89953a419fa1a722b) | Merge branch 'fix/vendor' into integ/all | SAV-SUPPORT |
| 38 | [f76021a4](https://github.com/Savas07/KenshiCoop/commit/f76021a42376e30c485202c0c6f15f575421fd88) | harness: host_pc_fight(+_mint) scenario + Test-HostPcFight oracle (join's copy of a fighting host PC) | SAV-SUPPORT |
| 39 | [73518a01](https://github.com/Savas07/KenshiCoop/commit/73518a017e1b3b1672b3e12d0b5adbfb7218d815) | harness: host_pc_fight - buffed fighters, fight after the join clock catch-up; gate on engaged-with-owner's-target | SAV-SUPPORT |
| 40 | [842160eb](https://github.com/Savas07/KenshiCoop/commit/842160ebb2d6aaec98e17429be26313ca30f61e0) | harness: host_pc_fight resident arm buffs defensive stats only (all-stat leaders chased fleeing bar NPCs at 120 u/s) | SAV-SUPPORT |
| 41 | [876ec41b](https://github.com/Savas07/KenshiCoop/commit/876ec41bf4bf24e632c8210a121d3659f11a9ed8) | harness: host_pc_fight resident arm - host orders each leader's bar victim onto it, drops victims that flee (chases were 70-120 u/s sprints, not melee) | SAV-SUPPORT |
| 42 | [337004a2](https://github.com/Savas07/KenshiCoop/commit/337004a2873dfb1e0f8077af596f0799d447e44b) | combat: resolve the streamed target through the drive's wire->local translation; judge wrong-target on bodies; no walk-order on arming/in-band squad copies | SAV-COMBAT |
| 43 | [5d845833](https://github.com/Savas07/KenshiCoop/commit/5d845833c0baedf62aade21fdba85be3c856072c) | combat: squad copies get a fresh re-issue budget per owner target and are re-ordered right after a snap | SAV-COMBAT |
| 44 | [65b81d53](https://github.com/Savas07/KenshiCoop/commit/65b81d53996b2e87b1a214952d4d20e0cdf98f7c) | combat: drop the immediate re-order after a squad-copy snap | SAV-COMBAT |
| 45 | [d2509ed3](https://github.com/Savas07/KenshiCoop/commit/d2509ed32ac7dc24f0a394374ba8f1445b115af3) | combat: restore the immediate re-order after a squad-copy snap | SAV-COMBAT |
| 46 | [3b1ad984](https://github.com/Savas07/KenshiCoop/commit/3b1ad984c901fd72b31f537dad4c992041927155) | docs: pitfalls 19 - squad-copy walk orders and per-target re-issue budget | SAV-SUPPORT |
| 47 | [77026f44](https://github.com/Savas07/KenshiCoop/commit/77026f447d7f1c7d0d5fafd090890b97effd3e58) | WIP fix/combat: host_pc_fight gate not yet stable after the fix | SAV-SUPPORT |
| 48 | [ef40acd5](https://github.com/Savas07/KenshiCoop/commit/ef40acd5c3b01f6e3c99ef2f8db42c9de160eaf0) | combat: an engaged squad copy is never walk-ordered or sustained-snapped mid-brawl | SAV-COMBAT |
| 49 | [50fcd027](https://github.com/Savas07/KenshiCoop/commit/50fcd027c95a8bea6dde7bc07fba9cf7b898e60c) | docs: pitfall 19 - engaged lock hold for squad copies; brawl-level engagement gate | SAV-SUPPORT |
| 50 | [4a2b69e2](https://github.com/Savas07/KenshiCoop/commit/4a2b69e27596e4bb7849f55dded1e34ef84bbba3) | Merge branch 'fix/combat' into integ/all | SAV-SUPPORT |
| 51 | [74049b22](https://github.com/Savas07/KenshiCoop/commit/74049b224724df52777184beeb947f7ea14c958f) | harness: mine_yield / mine_yield_join / bench_yield_join yield scenarios + Test-MineYield, Test-MachineSingle oracles | SAV-SUPPORT |
| 52 | [e5ad2ef0](https://github.com/Savas07/KenshiCoop/commit/e5ad2ef0a85fdecd88bbd9bb8b96d9c58fcec4ba) | fix: machine output simulated once (host); join skips operate() for driven workers and host-mirrored machines; [prod] operate diagnostics | SAV-PRODUCTION |
| 53 | [47e99af0](https://github.com/Savas07/KenshiCoop/commit/47e99af0cfec0482dd9c454eead24651b49a5c53) | harness: probe machine templates keyed on stringID (localised names broke placement); template dump; bench_yield_join input seeding, demoted to diagnostic | SAV-SUPPORT |
| 54 | [f80779ed](https://github.com/Savas07/KenshiCoop/commit/f80779edf95b9725890240114c211223422c2f66) | prod: gate only production+crafting operate on the host; prove crafting with bench_yield_join; re-pose lapsed machine copies | SAV-PRODUCTION |
| 55 | [51f3e178](https://github.com/Savas07/KenshiCoop/commit/51f3e178f37ceee5f98852c7e16612428cb3a3f8) | Merge branch 'fix/ore' into integ/all | SAV-SUPPORT |
| 56 | [d82a496d](https://github.com/Savas07/KenshiCoop/commit/d82a496d8ac1994f008818394728e82244a174aa) | world items: save-native ground items are structural baseline + pickup notice (upstream #44 #63 #66) | SAV-WORLD |
| 57 | [0c3def37](https://github.com/Savas07/KenshiCoop/commit/0c3def37aca7f58ecd40fb3bc799489428f61fcf) | fix(sync): replicate corpse inventories so loot is not duplicated (upstream #40) | SAV-CORPSE |
| 58 | [a6c97be0](https://github.com/Savas07/KenshiCoop/commit/a6c97be05cddc63fda4f8aade029a67814126cbf) | Fix #84: research started on a join pays no books and never reaches the host (protocol 63) | SAV-RESEARCH |
| 59 | [76975155](https://github.com/Savas07/KenshiCoop/commit/769751554310e1976e42155d4ac81457f9dd02e7) | Fix #74.1: join squad names, faction name and permanent jobs wiped by the host-authored save (protocol 64) | SAV-JOINSTATE |
| 60 | [0c90ef83](https://github.com/Savas07/KenshiCoop/commit/0c90ef83f1c7dcd1b38f04ac28d664afe18bb07e) | save: commit a transferred save only when it is complete, loadable and safely named (upstream #16) | SAV-SAVE |
| 61 | [32440706](https://github.com/Savas07/KenshiCoop/commit/32440706db1c86789dc467608ad7b9a4c61c3f6f) | inventory: refuse a loot of a unit that already left a container we author, and roll the loser's copy back (upstream #40 race) | SAV-XFER-RACE |
| 62 | [5fa67f96](https://github.com/Savas07/KenshiCoop/commit/5fa67f963f4b31d602b449e1f7cadde31fc4b52d) | Merge branch 'fix/lootrace' into integ/all | SAV-SUPPORT |
| 63 | [4bd312f8](https://github.com/Savas07/KenshiCoop/commit/4bd312f8b9d597a57024afebc6955cc750404668) | inventory: name the gear object that actually left the bag, and see a drop that empties the character (manual T8) | SAV-GEAR |
| 64 | [d5a933aa](https://github.com/Savas07/KenshiCoop/commit/d5a933aaaa78ccdeea7c9052fc4ee32f9f9fda84) | Merge branch 'fix/redrop' into integ/all | SAV-SUPPORT |
| 65 | [5c5f3886](https://github.com/Savas07/KenshiCoop/commit/5c5f3886f93769a3a795ac48a81eb73fb6723328) | drive: end the AI walk a JOIN recruit brings into the host's drive (manual recruit run desync) | SAV-RECRUIT |
| 66 | [6c908880](https://github.com/Savas07/KenshiCoop/commit/6c9088806b3c7496aa385360289a7f207dc386e7) | Merge branch 'fix/recruitrun' into integ/all | SAV-SUPPORT |
| 67 | [19497e70](https://github.com/Savas07/KenshiCoop/commit/19497e70c0a27c6642ae6285d79bf61ed4baefd2) | Trader lock + shop hand-over (protocol 65, manual T1) | SAV-TRADER |
| 68 | [41b09e3f](https://github.com/Savas07/KenshiCoop/commit/41b09e3fbb9206c2290744670574c4cda4d93405) | Merge branch 'fix/trader' into integ/all | SAV-SUPPORT |
| 69 | [3ec2551a](https://github.com/Savas07/KenshiCoop/commit/3ec2551af293eb0e55bbafe7e97854a976d59106) | Trader hand-over: touched shop lines with absolute counts (protocol 65) | SAV-TRADER |
| 70 | [996daa76](https://github.com/Savas07/KenshiCoop/commit/996daa7617e3169fd6fac1397dda0654ef0b2742) | Merge branch 'fix/trader' into integ/all | SAV-SUPPORT |
| 71 | [aa74a2b9](https://github.com/Savas07/KenshiCoop/commit/aa74a2b95041cbc0f1792a1b943784a7c56b3b33) | v0.58.0: one player at a time per trader + shop stock/money handed over; corpse loot no longer duplicates; join research pays books; save transfer safety; squad names/jobs kept; recruit and inventory desync fixes (protocol 65) | SAV-SUPPORT |
| 72 | [4b60185d](https://github.com/Savas07/KenshiCoop/commit/4b60185d3a2651463a13412c132fe87c52d5bdda) | README: credit the original author and this fork's maintainer | SAV-SUPPORT |
| 73 | [58514ea3](https://github.com/Savas07/KenshiCoop/commit/58514ea33538fca78e932abf44bb658d4e915236) | v0.58.1: backpacks a joining player gains are kept in the host's save; no duplicate bandits after a squad split; fewer NPC/carry desyncs (protocol 65) | SAV-IDENTITY-BAGS |

## Снимок веток

| Ref | SHA | Коммитов вне upstream main |
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

Источники получены прямым `git clone --bare`/`git fetch` публичных URL в refs/research без запуска кода и изменения удалённых репозиториев. Все сырые полные diff и per-branch history сохранены в `agent-core/refs_research_*.diff` и `*.history.txt`; `branch-summary.json` даёт file inventory. История orphan snapshot не означает дополнительные 73 уникальных изменения: для неё сравнивались деревья с Savas, а не только количество ahead.
