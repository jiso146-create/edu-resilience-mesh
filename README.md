# Ultra-Dense Mesh Flood Monitoring System
# 超高密度メッシュ型 浸水監視システム

**A low-cost, battery-free, self-healing flood detection network — fully open source.**
**低コスト・電池交換フリー・自己修復型の浸水監視ネットワーク。完全オープンソース。**

An educational DX project by [Bujutsu Souzou DIY & AI Lab], Japan.
[武術創造 DIY・AI研究所] による教育目的のDX探究プロジェクトです。

All software, firmware, circuit schematics, and educational materials are released under the MIT License.
すべてのソフトウェア・ファームウェア・回路図・教育用資料を MIT ライセンスのもとで公開しています。

> ⚠️ **Please read first / はじめに必ずお読みください**
>
> This is an educational project, intended for local residents as *supplementary reference only.* **Always follow the official disaster and evacuation information issued by your local government and national meteorological agency.** The output of this system must never take priority over official warnings.
>
> これは学校の学習プロジェクトです。地域のみなさんには「参考情報」としてご覧いただくことを想定しています。**実際の避難の判断は、必ずお住まいの自治体および気象庁の情報を最優先してください。** 本システムの表示や通知が、公式の避難情報に優先することは一切ありません。

---

# English

## Why this project exists

Every year, floods take lives in regions that cannot afford to watch the water. Commercial flood-monitoring systems are often too expensive to deploy densely, and — more importantly — too expensive to *keep running*. The real enemy of large-scale sensing is not the price of a sensor. It is the **recurring cost**: cellular data fees, battery replacement, server bills, and endless maintenance visits.

This project is our attempt, as students, to answer one question:

**How cheaply, how durably, and how maintenance-free can you keep *waiting* for a rare disaster?**

Because that is what flood monitoring really is: staying ready through the 99.9% of the time when nothing happens.

## Design philosophy

**1. "Doing nothing" is the normal state.**
Most of the time, this system detects nothing — and that is correct. The challenge is not sensing water; it is *waiting* for it, cheaply and indefinitely, without maintenance.

**2. Subtraction, not addition — decide what NOT to protect.**
Trying to protect everything makes a device expensive and fragile. We do not measure continuous depth. We detect only two thresholds by the physical height of two electrodes: **5 cm (caution)** and **15 cm (danger)**. Beyond that, the device itself goes underwater and stops responding — and we treat that **silence (LOST) as the strongest danger signal of all (black).** A sensor going quiet becomes its final, most eloquent message.

**3. Sleep by default, wake on water.**
The electrodes carry no current in normal conditions. The microcontroller wakes only to send a **heartbeat every 2 hours**, then sleeps. This interval balances energy savings, speed of LOST detection, and network congestion.

**4. Two-tier monitoring escalation.**
A node reaching **5 cm autonomously switches to ~10-minute reporting** (local escalation). The central backend can also push an **"increase frequency" command to all nodes at once** ahead of a forecast storm (central escalation). *Honest limitation:* because nodes sleep to save power, a pushed command may be delayed by up to one heartbeat interval. This responsiveness-vs-power tradeoff is a problem we are still working on.

**5. Kill the recurring communication cost by design.**
Instead of cellular (LPWA / LTE) links with yearly fees, nodes relay to each other over **Thread (IPv6-based self-healing mesh)**, and a border-router parent bridges to **existing Wi-Fi** at a school or community hall. The backend runs on the **free tier of Google Apps Script**. This drives the *ongoing* operating cost close to zero. If one node is lost, neighbors automatically rebuild an alternate route.

**6. Optimize for your land — power that matches the climate.**
Power comes only from a small solar panel and a supercapacitor (EDLC) — no chemical batteries, no leakage or fire risk, very long cycle life, tolerant of heat and cold. This depends on sunlight. Our own prototype is tuned for a sunny, low-rainfall-variance region (we developed it in Okayama, a part of Japan known as the "Land of Sunshine"). **That is exactly the point: the optimal engineering answer changes with your environment.** In a monsoon climate with long overcast rains — or anywhere with different sunlight, terrain, or infrastructure — the power design must be rethought. This open source exists so you can retune it for your own land.

**7. The microcontroller is disposable; the standard is the asset.**
By building on the industry-standard **Thread / IEEE 802.15.4**, a better chip in the future can replace the nodes while the network stays intact. Open source and open standards protect this "freedom to migrate."

## Architecture

```
[ Each gutter / school route — child node ]
  Solar panel -> Supercapacitor (EDLC) -> Buck-Boost charge management
        |
        v
  ESP32-C6 (Thread) + electrode water detection (5 cm / 15 cm)
        |
        v  autonomous mesh relay, ~100-200 m spacing
[ node ] -> [ node ] -> ... -> [ School / community hall = Border Router ]
        |
        v  existing Wi-Fi
  Google Apps Script (backend + API)
        |
   +----+---------------------------+
   v                                v
 Dashboard (4-color map)     Public messaging channel (alerts)
```

Status colors: **blue** (normal) / **yellow** (5 cm) / **red** (15 cm) / **black** (no response = LOST = highest danger).

## Who might find this useful

We believe this design is most valuable not where money is abundant, but where it is scarce — regions facing recurring floods without the budget to sustain expensive monitoring. If you are a researcher, an NGO, a teacher, or a community maker working on **low-cost flood early warning**, please take this, break it, and rebuild it for your own conditions. That is what it is for.

## Relationship to existing efforts

Japan's national government (MLIT) runs a large-scale "One-Coin Flood Sensor" field trial. This project does not compete with it and is not a replacement. It is a way to ask: *"The nation answered this challenge one way — how would we answer it?"* Public programs and grassroots ingenuity differ in speed, coverage, and accountability, and understanding that difference is part of the lesson.

## Status

An educational prototype under active development. Expect rough edges. Contributions, forks, and regional adaptations are warmly welcomed.

## Disclaimer

Provided for educational purposes. No guarantee is made as to accuracy, availability, or fitness for any purpose. The authors accept no liability for any damages arising from use of this system, including its use in evacuation decisions. Always follow official information from your local authorities and meteorological agency.

## License

All materials in this repository — software, firmware, circuit schematics, and educational curricula, manuals, and documentation — are released under the **MIT License**. See [LICENSE](./LICENSE).

---

# 日本語

## このプロジェクトの目的

洪水は毎年、水を監視する余裕のない地域で命を奪っています。市販の浸水監視システムは、密に設置するには高価すぎ、そして何より「動かし続ける」コストが高すぎます。大規模なセンシングの本当の敵は、センサ本体の値段ではありません。通信費・電池交換・サーバ費用・終わりのない保守——毎年かかり続ける**ランニングコスト**です。

このプロジェクトは、私たちが、たったひとつの問いに答えようとした試みです。

**めったに起きない災害を、どれだけ安く、頑丈に、メンテナンスなしで「待ち構え続けられるか」。**

なぜなら浸水監視とは、まさにそれ——何も起きない99.9%の時間を、いかに備えたまま過ごせるか、の戦いだからです。

## 設計思想

**1. 「何も検知しないこと」が正常である。**
このシステムは大半の時間、何も検知しません。そしてそれが正しい状態です。難しいのは水を感じることではなく、それを安く、無期限に、メンテナンスなしで「待つ」ことです。

**2. 足し算ではなく引き算 ― 守らないものを決める。**
すべてを守ろうとすると、高価で壊れやすくなります。水位は連続値では測らず、二本の電極の高さの差で **5cm(注意)** と **15cm(危険)** の二段階だけを検知します。それを超える冠水では、デバイス自身が水没して応答を止めます。私たちはこの**「応答なし(ロスト)」という状態そのものを、最大の危険信号(黒)として扱います。** センサが黙ることが、最後の、そして最も雄弁な通信になります。

**3. 普段は寝て、水が来たら起きる。**
電極は平常時には通電せず、消費電力はほぼゼロです。マイコンは **2時間おきにハートビート(生存確認)** だけを送り、また眠ります。この間隔は、電力の節約・ロスト検知の速さ・通信の混雑という三つのバランスで決めました。

**4. 二段構えの監視モード。**
**5cm に達した子機は、自ら観測間隔を約10分に切り替えます**(自律格上げ)。さらに、大雨の予報などに応じて、中央のサーバから **全子機へ「観測頻度を上げよ」という指令を一斉に下ろす**こともできます(中央格上げ)。*正直な限界:* 省電力のため子機が眠っている間は、指令の反映に最大でハートビート間隔ぶんの遅れが生じます。この「即応性と省電力のトレードオフ」は、私たちが今も向き合っている課題です。

**5. 通信のランニングコストを「構造」で消す。**
毎年課金される携帯回線(LPWA / LTE)を使わず、子機どうしが **Thread(IPv6ベースの自己修復メッシュ)** で中継し合い、親機(Border Router)が学校や公民館の **既存 Wi-Fi** に接続します。バックエンドは **Google Apps Script の無料枠**を使います。こうして運用の維持費を構造的にほぼゼロにしました。1台が失われても、周囲の子機が別の中継ルートを自動で組み直します。

**6. 土地に合わせて最適化する ― 気候に合った電源を。**
電源は小型の太陽光パネルとスーパーキャパシタ(EDLC)のみ。化学電池を使わないため液漏れ・発火のリスクがなく、超高サイクル寿命で、高温・低温にも強い構成です。これは日照に依存します。私たち自身の試作機は、日照が多く天候の振れが小さい地域に合わせて調整しています(開発地は「晴れの国」として知られる岡山です)。**ここが肝心です。最適な工学的答えは、その土地の環境で変わります。** 連日曇天の雨が続くモンスーン気候、あるいは日照・地形・インフラの異なるあらゆる土地では、電源設計を見直す必要があります。このオープンソースは、あなたが自分の土地に合わせて調整し直せるように存在します。

**7. マイコンは消耗品、規格は資産。**
業界標準の **Thread / IEEE 802.15.4** の上に築いたことで、将来より優れたチップが出れば、ネットワークはそのままに子機だけを載せ替えられます。オープンソースと標準規格は、この「未来への乗り換え可能性」を守るためにあります。

## システム構成

```
[ 各側溝・通学路 — 子機 ]
  太陽光パネル -> スーパーキャパシタ(EDLC) -> 昇降圧・充放電管理
        |
        v
  ESP32-C6 (Thread) + 電極による水位検知 (5cm / 15cm)
        |
        v  100〜200m間隔の自律メッシュリレー
[ 子機 ] -> [ 子機 ] -> ... -> [ 学校・公民館 = Border Router ]
        |
        v  既存Wi-Fi
  Google Apps Script (バックエンド + API)
        |
   +----+---------------------------+
   v                                v
 ダッシュボード(4色マップ)     住民向け通知チャンネル(アラート)
```

状態の色: **青**(平常) / **黄**(5cm) / **赤**(15cm) / **黒**(応答なし = ロスト = 最大の危険)。

## 誰の役に立つか

この設計が最も価値を持つのは、お金が潤沢な場所ではなく、乏しい場所——高価な監視を維持する予算がないまま、毎年洪水に直面する地域だと私たちは考えています。**低コストの洪水早期警報**に取り組む研究者、NGO、教員、地域の作り手の方は、どうかこれを持っていき、壊し、自分の土地の条件に合わせて作り直してください。そのためのオープンソースです。

## 既存の取り組みとの関係

日本政府(国土交通省)は「ワンコイン浸水センサ」の大規模な実証実験を進めています。本プロジェクトはそれと競うものでも、代わるものでもありません。**「同じ課題に、国はこう答えた。では私たちはどう答えられるか」** を考えるための取り組みです。公的な取り組みと草の根の工夫は、速さ・網羅性・責任の所在という点で役割が違う——それを理解することも学びの一部です。

## 状態

開発中の教育用プロトタイプです。粗削りな点があります。貢献・フォーク・各地域向けの改変を歓迎します。

## 免責事項

本成果物は教育目的で提供されます。正確性・可用性・特定目的への適合性を保証しません。作者は、本システムの使用(避難判断への利用を含む)から生じるいかなる損害についても責任を負いません。避難に関する判断は、必ず自治体および気象庁の公式情報に従ってください。

## ライセンス

本リポジトリに含まれるすべての成果物(ソフトウェア・ファームウェア・回路図・教育用カリキュラム・マニュアル・文書)を **MIT ライセンス**のもとで公開します。詳細は [LICENSE](./LICENSE) を参照してください。
