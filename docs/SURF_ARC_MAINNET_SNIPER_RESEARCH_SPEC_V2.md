# Arc Mainnet Meme Sniper — SURF Research Specification v2

調査基準日: 2026-09-18  
目的: Arc Mainnet上でミームコインSniper戦略を研究・実装する価値があるかを、再現可能なMainnet証拠と実行可能損益で判定する。

---

## 0. 最重要原則

今回の目的は「勝てるSniper戦略をそれらしく提案する」ことではない。

必ず次の順序で調査する。

1. Chain identity
2. Mainnet observability
3. Permissionless market existence
4. Launch / Pool / BUY / SELL complete decode
5. 外部BOTが参入可能になる最速時点
6. Position-consistent executable return
7. Right tail
8. Safety / loser filter possibility
9. Data collection PoC / Execution PoCの可否

途中GateがFAILまたは停止条件付きUNKNOWNなら、後続分析を行わない。

ML、feature engineering、TP/SL最適化、BOT実装は今回行わない。

推測は禁止。不明はUNKNOWN。

---

# 1. 情報源の優先順位

1. Circle / Arc公式Docs
2. Arc公式Explorer / Arc Mainnet RPC
3. Protocol公式Docs / GitHub / deployment情報
4. Verified contract source
5. Mainnet transaction / receipt / logs
6. Explorer / Indexer
7. 信頼できる第三者資料
8. X / コミュニティ情報

第三者記事だけからMainnet稼働、contract address、permissionless利用可否を断定しない。

TestnetとMainnetを混同しない。

各重要claimはEvidence Ledgerへ記録する。

---

# 2. 調査cutoffを固定

調査開始時に必ず以下を固定する。

```text
cutoff_timestamp_utc
cutoff_block_number
cutoff_block_hash
chain_id
rpc_endpoint
rpc_observation_time_utc
```

加えて、可能なら母集団開始点も固定する。

```text
population_start_block
population_start_timestamp_utc
```

直近Launchのうち、評価Horizonがcutoffを越えるものは「未達」とせずRIGHT_CENSOREDにする。

---

# 3. Phase進行規則

| Phase結果 | 後続Phase |
|---|---|
| PASS | 進行 |
| PARTIAL | 必須項目が全て確認済みの場合のみ進行 |
| FAIL | 停止 |
| UNKNOWN | 原則停止 |
| Provider固有欠測 | 別providerで再検証後に再判定 |

UNKNOWNを推測でPARTIAL/PASSに変更しない。

---

# PHASE 0 — Chain Identity / Mainnet Observability

## 4. Chain identity

確認すること:

- 公式Chain名称
- 公式Chain ID
- 公式Mainnet Docs
- 公式HTTP RPC
- 公式WSS RPC
- 公式Explorer
- latest blockが継続的に進むこと
- transaction receipt取得
- logs取得
- block取得
- chain finality仕様
- reorg特性

以下を区別する。

```text
official_genesis_block
earliest_retrievable_block_from_selected_rpc
current_latest_block
```

earliest retrievable blockをgenesisと同一視しない。

## 5. RPC/WSS実確認

最低限確認:

- eth_chainId
- eth_blockNumber
- eth_getBlockByNumber
- eth_getTransactionReceipt
- eth_getLogs
- eth_getBalance
- eth_call
- eth_estimateGas
- WSS newHeads
- WSS logs

pending transactionについて:

```text
unsupported
accepted_empty
partial_visibility
public_mempool_visible
validator_private_only
not_reproducible
```

のいずれかへ分類する。

単にsubscriptionがacceptedされたことを「public mempool可視」としない。

## 6. Gas asset

以下を前提にせず検証する。

- native gas asset
- decimals
- eth_getBalanceの意味
- ERC-20 USDC address
- native balanceとERC-20 balanceOfの関係
- transaction value
- routerが受け取るnative asset
- swap input asset
- gas debit

Native USDCとERC-20 USDCを同一と推測しない。

## Phase 0 PASS条件

以下すべて確認:

1. 公式Mainnet根拠
2. Chain ID一致
3. RPC応答
4. latest block進行
5. receipt取得
6. getLogs取得

WSSが一部不足しても、HTTPでChain identityを完全確認できればPARTIAL進行可。
上記1〜6のいずれかがUNKNOWNなら停止。

---

# PHASE 1 — Market Exists

## 7. Venue discovery

Arc Mainnet上で以下を探索:

- Memecoin Launchpad
- Permissionless Token Launcher
- Bonding Curve
- Fair Launch
- one-click ERC20 launcher
- 独自AMM
- Uniswap V2/V3/V4へ直接Poolを作るlauncher
- その他permissionless新規token市場

証拠レベル:

```text
ANNOUNCED
TESTNET
MAINNET_CONTRACT_DEPLOYED
MAINNET_UI_LIVE
PERMISSIONLESS_CREATE_VERIFIED
POOL_VERIFIED
TRADE_VERIFIED
BUY_SELL_VERIFIED
INACTIVE
UNKNOWN
```

LIVEという一語で済ませない。

各Venueで保存:

- name
- official URL
- official X
- Factory
- Router
- Token implementation
- Pool / PoolManager
- verified source
- proxy implementation
- upgradeability
- first observed Mainnet tx
- most recent observed tx
- last token creation
- last BUY
- last SELL
- observation cutoff block/time

## 8. Gate A母集団

候補Venueごとに、固定block範囲内の全Permissionless Launchを母集団にする。

原則:

```text
population =
selected Factory / Launch contractが
[population_start_block, cutoff_block_number]
で生成した全permissionless launch
```

成功例だけを手動選択しない。

## Gate A PASS基準

最低1 Venueについて:

- distinct permissionless Launch >= 10
- 観測期間 >= 24h、ただしMainnet公開から24h未満の場合は利用可能全期間
- 完全に異なる3 Launch以上で
  token creation + pool/liquidity + successful BUY + successful SELL
  を確認
- cutoff直近48h以内、またはMainnet公開からの全期間内に新規activityあり

PARTIAL:
- Launch 3〜9件
- complete BUY/SELL sequenceは3件あるが観測期間不足
- 市場が新しすぎて率評価不能

FAIL:
- permissionless Launch 0
- contract/UIのみで実Launchなし
- token creationはあるが実BUY/SELL市場が成立しない

UNKNOWN:
- RPC/Explorer欠測で母集団自体を構築できない

FAIL/UNKNOWNなら停止。

---

# PHASE 2 — Complete Decode

## 9. 最有望Venueを1つ選ぶ

この段階では1 Venueだけ。

選定根拠:

- Launch数
- 最近のactivity
- liquidity
- Contract透明性
- BUY/SELL実績
- event detection容易性
- historical data取得容易性

## 10. 最低3 Launchの完全sequence

最低3つの異なるPermissionless Launchで:

```text
Launch
→ Token creation
→ Pool creation
→ Initial liquidity
→ First successful BUY
→ First non-creator BUY
→ First successful SELL
```

を再構築。

保存:

- block number
- block hash
- block timestamp
- tx index
- tx hash
- receipt status
- call depth / trace address
- log index
- emitting contract
- method
- calldata
- event
- creator
- trader
- token
- quote asset
- pool
- base amount raw
- quote amount raw
- decimals
- gas used
- effective gas price
- protocol/router/transfer fee

tx hash文字列順を経済的orderに使用しない。

## 11. BUY/SELL decode integrity

BUY Entry価格はBUY transactionからのみ取得。
SELL Exit価格はSELL transaction / deterministic SELL simulationからのみ取得。

```text
execution_price = quote_amount / base_amount
```

ただしraw integer amountとdecimalsを保存する。

最低10 swap、または存在する全swapが10未満なら全件について独立照合:

- decoded event/call amount
- wallet balance delta
- Transfer logs
- pool reserve/state change
- trace value flow

ERC-20標準transferでfee/taxが無い部分はraw integer単位で整合すること。
差分がある場合は丸め誤差で曖昧化せず、protocol fee / transfer tax / router flow等の理由を説明できること。

不明な不整合が1件でも残れば該当transactionはUNVERIFIED。

## Gate B PASS基準

- 3 Launch以上のcomplete sequence
- Launch/Pool/BUY/SELL identity decode 100%
- 独立amount照合対象の>=95%が整合
- 残り<=5%も既知fee/tax等で説明可能
- unexplained side mismatch = 0
- unexplained token orientation mismatch = 0

PARTIAL:
- identityは成立するがtrace/archive不足で一部amountを独立照合不能

FAIL:
- BUY/SELL sideを安定して判定不能
- token orientation不明
- amountの重大不整合
- SELL pathをdecode不能

FAILなら停止。

---

# PHASE 3 — External Executability / Right Tail

# 12. 検知モードを分離する

「first externally executable BUY after detection」を単一概念にしない。

最低以下を別Scenarioとして扱う。

## D0 — Public pending

Launch transactionを包含前にpublic RPC/WSSで観測できる場合のみ。

## D1 — WSS logs

providerがLaunch/Pool logを配信した時点。

## D2 — newHeads

Launchを含むblock headerを受信し、その後logs/receipt/block bodyを取得・decode。

## D3 — Confirmed receipt / HTTP

Launch receipt確認後に発注。

Private order flowは一般外部BOTから再現不能なら別枠にし、D0扱いしない。

各live観測でローカルmonotonic clockを使い:

```text
t_provider_received
t_decoded
t_decision_ready
t_signed
t_rpc_accepted
t_included
```

を記録。

historical receiptだけから
「当時WSSならこの価格で間に合った」
とは断定しない。

live latencyデータが無ければGate EはUNKNOWN/PARTIAL。

---

# 13. Position-consistent notional

notional Qは:

```text
Entry時にwalletからSwap用として支出するquote amount
```

と定義する。

固定Q:

- $1
- $5
- $10
- $25
- $50

USD換算できないquoteの場合は、変換source/timeを記録。

必要に応じて追加:

- quote-side real reserveの1%
- 5%
- 10%

TVL、virtual liquidity、両側TVLを「liquidity」と曖昧に扱わない。

## BUY

```text
B(Q) =
QをBUYに使った結果、
walletが実際に受け取るbase token net amount
```

fee-on-transferがある場合gross outputではなくwallet net increaseを使用。

## SELL

Exit時点tでは必ず:

```text
B(Q)全量
```

をSELLする。

$5 Entryの後に「$5相当だけSELL」のような独立notional評価は禁止。

---

# 14. Wallet損益の構成要素

各Qで保存:

```text
entry_swap_quote_debit
entry_gas_quote_equivalent
entry_protocol_fee
entry_router_fee
entry_transfer_fee

base_gross_output
base_wallet_net_increase = B(Q)

exit_base_debit
exit_swap_quote_credit
exit_gas_quote_equivalent
exit_protocol_fee
exit_router_fee
exit_transfer_fee
```

feeがwallet deltaに既に含まれる場合、Net Return計算で二重加算しない。

protocol fee等は説明用componentとして保存し、
最終損益はwallet economic deltaを基準にする。

---

# 15. Return定義

## Market Round-trip Return

gasを除外したswap経済性:

```text
R_market(t,Q)
=
exit_swap_quote_credit(t,Q)
/
entry_swap_quote_debit(Q)
- 1
```

## Net Wallet Return — 主指標

```text
R_net(t,Q)
=
(
exit_swap_quote_credit(t,Q)
- exit_gas_quote_equivalent(t,Q)
)
/
(
entry_swap_quote_debit(Q)
+ entry_gas_quote_equivalent(Q)
)
- 1
```

Entry/Exit feeがswap debit/creditに既に反映される場合は再加算しない。

gas assetがquoteと異なる場合のみ、同時点の明示された変換sourceでquote換算する。

---

# 16. Historical SELL simulation hierarchy

過去の時点tでB(Q)を売却できたかは、以下の優先順位で判定。

1. archive stateに対するprotocol quote / eth_call
2. 保存されたpool stateから公式AMM数式でdeterministic simulation
3. trace/state replay
4. それらが不可能ならUNKNOWN

他人の小額SELL execution priceだけを
「自分のB(Q)全量もその価格で売れる」
根拠にしない。

実際の他者SELLはdecoder validationには使用してよいが、
notional-specific executable returnの代用にはしない。

---

# 17. UNSellable taxonomy

SELL不能を欠測として削除しない。

以下へ分類:

```text
CONTRACT_REVERT
INSUFFICIENT_LIQUIDITY
TRANSFER_RESTRICTION
BLACKLISTED
MAX_TX_RESTRICTION
NO_ROUTE
QUOTE_FAILED
LIQUIDITY_REMOVED
RPC_DATA_MISSING
ARCHIVE_STATE_UNAVAILABLE
UNKNOWN_REASON
```

経済的SELL不能:

- CONTRACT_REVERT
- INSUFFICIENT_LIQUIDITY
- TRANSFER_RESTRICTION
- BLACKLISTED
- MAX_TX_RESTRICTION
- NO_ROUTE
- LIQUIDITY_REMOVED

はNet Wallet Return集計から除外せず、原則「全額回収不能」として別途損失caseへ計上する。

RPC_DATA_MISSING / ARCHIVE_STATE_UNAVAILABLEはeconomic failureと混同せず欠測扱い。

---

# 18. Phase 3母集団

Right-tail分析対象は:

```text
selected Venue
× fixed population block range
× 全permissionless Launch
```

または事前に固定した機械的包含条件を満たす全Launch。

勝者だけの手動sampleは禁止。

## 技術的除外可能

事前登録された場合のみ:

- test tokenと公式に確認できる
- creator-only専用launchで外部参加不能
- chain reorgでcanonical chainから消失
- ABI/bytecodeが完全欠損しdecode不能
- 対象quote asset外

## 除外してはいけない

- BUYできるがSELL不能
- honeypot
- liquidity撤去
- Horizon内取引なし
- reverted sell
- token taxが極端
- price impactが巨大
- scam/rug

これらは戦略上の失敗例。

---

# 19. Right censoring

Launch時刻 + Horizon > cutoffの場合:

```text
RIGHT_CENSORED
```

未達=falseにしない。

Horizon:

- 5m
- 15m
- 1h
- 6h
- 24h

各Horizonでdenominatorは:

```text
uncensored
AND
execution-evaluable
```

を明示。

economic UNSellableはdenominatorに残す。
provider/archive欠測は別にmissingとして表示。

---

# 20. MFE / MAE

各Qについて主指標はNet Wallet Return。

```text
Net_MFE_T(Q) = max R_net(t,Q)
Net_MAE_T(Q) = min R_net(t,Q)
```

市場構造観察用に:

```text
Market_MFE_T(Q) = max R_market(t,Q)
Market_MAE_T(Q) = min R_market(t,Q)
```

も保存。

倍率到達は主としてNet Wallet Returnで判定:

```text
1.5x = R_net >= 0.5
2x   = R_net >= 1
3x   = R_net >= 2
5x   = R_net >= 4
10x  = R_net >= 9
20x  = R_net >= 19
50x  = R_net >= 49
100x = R_net >= 99
```

各倍率で:

- count
- denominator
- rate
- Wilson 95% CI
- time-to-hit
- notional
- sellability
- price impact
- missing count
- censored count

を報告。

---

# 21. Gate C — Executability

Primary notional:

```text
Q_primary = $5
```

Secondary:

$1 / $10 / $25 / $50

Gate C PASS:

- uncensored/evaluable Launch >= 20
- $5 Entry成功率 >= 80%
- $5で同一B(Q)全量SELL成功率 >= 80%
- $10 round-trip成功率 >= 60%
- unexplained revert rate <= 10%

PARTIAL:

- evaluable N < 20
- $5 round-trip 50〜79%
- $10結果不足

FAIL:

- $5 round-trip < 50%
- widespread honeypot / no-route / liquidity-removalにより往復不能

これは「利益が出る」Gateではなく「現実に取引可能」Gate。

---

# 22. Gate D — Right Tail

Gate Dは「勝てる戦略」ではなく、
大きな右裾が実行可能価格で存在するかを判定する。

Primary = $5 Net Wallet Return。

PASS:

- uncensored/evaluable N >= 50
AND
- >=5xが2件以上
  OR
- >=10xが1件以上

PARTIAL:

- N 20〜49で>=3xを1件以上確認
- またはN>=50で>=3xは存在するが5x条件未達

FAIL:

- N>=50
AND
- >=3xが0件

UNKNOWN:

- N<20
- archive/state不足で実行可能Exitを評価不能

Gate D PASSは収益性PASSではない。
単にLottery/Runner型戦略を研究する最低限の右裾があるという意味。

---

# 23. Gate E — Detect Fast Enough

履歴推測だけではPASSにしない。
live observation必須。

最低20 live Launch、または市場全件が20未満なら観測可能全件を対象。

各Launchで:

- t_provider_received
- t_decoded
- t_decision_ready
- block inclusion ordering
- first non-creator BUY block/index
- first external BUY after decision-ready

を保存。

Primary detection modeは、
利用可能な最速の「一般外部BOTが再現可能なpublic経路」とする。

Gate E PASS:

- live N >= 20
- p90(t_decision_ready - t_provider_received) < median block interval
- >=70%のLaunchでdecision-ready後にも外部BUY機会が残る
- そのうち>=50%で少なくとも次blockまで外部BUYが継続

PARTIAL:

- N不足
- timingは良好だがpending visibility等がprovider依存
- shadow transaction buildまでしか検証できない

FAIL:

- 一般外部BOTがLaunchを知る頃には大半の買い機会が終了
- public経路では再現不能

receipt/blockだけのhistorical dataではGate EをPASSにしない。

---

# 24. Safety / loser filter候補

Gate A〜DがFAILなら、ML/filter研究へ進まない。

進める場合だけ、Launch時点以前または指定時間までに因果的に利用可能なfeatureを整理。

候補:

- creator history
- deployer history
- funding source
- wallet age
- previous launches
- previous rugs
- initial liquidity
- token bytecode properties
- owner/admin
- proxy/upgradability
- mint privilege
- blacklist/pause
- transfer/sell tax
- creator allocation
- holder concentration
- initial buyers
- unique buyers
- buy/sell count
- buy/sell imbalance
- trader growth
- volume
- return
- realized volatility
- HHI
- top1 buyer share

各featureに:

```text
earliest_causal_availability
source
live retrieval latency
historical reconstructability
```

を付ける。

---

# 25. Evidence Ledger

必須成果物。

最低列:

```text
claim_id
phase
gate
claim
result
chain_id
block_number
block_hash
tx_hash
tx_index
log_index
contract
implementation_contract
source_type
source_url
abi_source
abi_version_or_commit
observation_time_utc
rpc_endpoint_class
raw_evidence_path
reproduction_method
caveat
```

proxyの場合:

- implementation address
- admin
- upgrade mechanism
- implementation確認block

を保存。

---

# 26. Raw evidence保存要件

少なくとも以下を保存可能な形で整理:

- RPC request/response
- block
- transaction
- receipt
- logs
- trace（使用した場合）
- contract ABI
- verified source reference
- proxy implementation
- token metadata
- pool state snapshot
- quote/simulation input-output
- local observation timestamps

同じ調査を別担当者が再実行できること。

---

# 27. 最終Gate一覧

## Gate A — Market Exists
PASS基準: Section 8。

## Gate B — Decode Possible
PASS基準: Section 11。

## Gate C — Executable
PASS基準: Section 21。

## Gate D — Right Tail Exists
PASS基準: Section 22。

## Gate E — Detect Fast Enough
PASS基準: Section 23。

Gateごとに:

```text
PASS
PARTIAL
FAIL
UNKNOWN
```

を出す。

---

# 28. Final Verdict

最終判断は三択。

## NO_GO

Chain / Market / DecodeのいずれかがFAIL、
または研究継続を正当化する証拠がない。

## DATA_COLLECTION_POC_ONLY

Market / Decodeは成立するが、

- sample不足
- latency不明
- right-tail評価不足
- historical state不足

などでExecution PoCへ進めない。

次工程はデータ収集だけ。

## PROCEED_TO_EXECUTION_POC

最低条件:

- Gate A PASS
- Gate B PASS
- Gate C PASS
- Gate D PASS
- Gate E PASSまたは強いPARTIALで不足点が明確

このVerdictは
「利益が出るBOTが確定」
を意味しない。

少額Execution PoCへ進む価値がある、という意味。

---

# 29. 最終出力形式

1. Executive Summary
2. Frozen Cutoff
3. Phase 0 Chain Verification
4. Phase 1 Market Population
5. Venue Evidence Levels
6. Selected Venue
7. Complete Launch Sequences
8. BUY/SELL Decoder Validation
9. Detection Modes
10. Position-consistent Execution Model
11. Round-trip Sellability by Notional
12. Market MFE/MAE
13. Net Wallet MFE/MAE
14. Right-tail counts + Wilson 95% CI
15. UNSellable / Missing / Censored breakdown
16. Token Safety
17. Creator / Wallet Data Availability
18. Gate A〜E
19. Final Verdict
20. Unknowns
21. Evidence Ledger
22. Sources

---

# 30. 最重要禁止事項

以下は禁止。

- TestnetをMainnetとして扱う
- announcedをlive扱い
- contract deployedだけでmarket exists判定
- 成功例だけの手動sampling
- 他人SELL価格をBUY Entryに使用
- EntryとExitで異なる数量を比較
- spot priceだけでMFE計算
- unsellableを欠測として削除
- provider欠測をrug扱い
- right-censoredを未達扱い
- historical receiptだけからWSS latencyを断定
- fee/gasの二重計上
- future dataをEntry featureへ混入
- N不足で到達率を断定
- UNKNOWNを都合よくPASSへ変更

最初に事実と実行可能性を確定し、その後だけ戦略研究へ進むこと。
