Ultra-Dense Mesh Flood Monitoring System
超高密度メッシュ型 浸水監視システム
A low-cost, maintenance-minimized, self-healing flood detection network — fully open source. 低コスト・保守の手間を最小化・自己修復型の浸水監視ネットワーク。完全オープンソース。

An educational DX project by [Bujutsu Souzou DIY & AI Lab], Japan. [武術創造 DIY・AI研究所] による教育目的のDX探究プロジェクトです。

All software, firmware, circuit schematics, and educational materials are released under the MIT License. すべてのソフトウェア・ファームウェア・回路図・教育用資料を MIT ライセンスのもとで公開しています。

⚠️ Please read first / はじめに必ずお読みください

This is an educational project, intended for local residents as supplementary reference only. Always follow the official disaster and evacuation information issued by your local government and national meteorological agency. The output of this system must never take priority over official warnings.

これは学校の学習プロジェクトです。地域のみなさんには「参考情報」としてご覧いただくことを想定しています。実際の避難の判断は、必ずお住まいの自治体および気象庁の情報を最優先してください。 本システムの表示や通知が、公式の避難情報に優先することは一切ありません。

English
Why this project exists
Every year, floods take lives in regions that cannot afford to watch the water. Commercial flood-monitoring systems are often too expensive to deploy densely, and — more importantly — too expensive to keep running. The real enemy of large-scale sensing is not the price of a sensor. It is the recurring cost: cellular data fees, battery replacement, server bills, and endless maintenance visits.

This project is our attempt, as students, to answer one question:

How cheaply, how durably, and with how little maintenance can you keep waiting for a rare disaster?

Because that is what flood monitoring really is: staying ready through the 99.9% of the time when nothing happens.

Design philosophy
1. "Doing nothing" is the normal state. Most of the time, this system detects nothing — and that is correct. The challenge is not sensing water; it is waiting for it, cheaply and indefinitely, with as little maintenance as possible.

2. Subtraction, not addition — decide what NOT to protect. Trying to protect everything makes a device expensive and fragile. We do not measure continuous depth. We detect only two thresholds by the physical height of two electrodes: 5 cm (caution) and 15 cm (danger). Beyond that, the device itself goes underwater and stops responding — and we treat that silence (LOST) as the strongest danger signal of all (black). A sensor going quiet becomes its final, most eloquent message.

3. Fail-safe by design — silence is the alarm (normally-closed thinking). Most flood sensors are built "normally-open": they stay silent and speak only when water arrives. But that silence cannot tell "safe" apart from "broken." We invert this. Our nodes send a proof-of-life heartbeat during normal times (a "normally-closed" mindset), and treat its absence — the silence itself — as the highest alarm. Even when a device is submerged and stops working, its death becomes the information: "the water has exceeded the danger level here." This is exactly why the hardware is allowed to drown (IP56-class housing is enough) and we spend nothing on excessive waterproofing. When it fails, it fails toward the alarm side — the same fail-safe philosophy used in railway signaling and elevator safety circuits. The opposite — a sensor that dies quietly and no one notices until the next flood — is what we refuse to build.

4. Sleep by default, wake on water. The electrodes carry no current in normal conditions. The microcontroller wakes only to send a heartbeat every 2 hours, then sleeps. This interval balances energy savings, speed of LOST detection, and network congestion. This extremely low duty cycle is also what keeps the power requirement tiny — and, as a result, what makes almost any power source viable (see #8).

5. Two-tier monitoring escalation. A node reaching 5 cm autonomously switches to ~10-minute reporting (local escalation). The central backend can also push an "increase frequency" command to all nodes at once ahead of a forecast storm (central escalation). Honest limitation: because nodes sleep to save power, a pushed command may be delayed by up to one heartbeat interval. This responsiveness-vs-power tradeoff is a problem we are still working on.

6. Kill the recurring communication cost by design. Instead of cellular (LPWA / LTE) links with yearly fees on every unit, nodes relay to each other over Thread (IPv6-based self-healing mesh), and a border-router parent bridges to existing infrastructure. Many children are bundled under a single parent, so only the parent needs an uplink — driving the ongoing operating cost close to zero. The parent bridges via existing Wi-Fi (school, community hall) where available, or LTE-M where it is not. If one node is lost, neighbors automatically rebuild an alternate route.

7. Put the parent on existing infrastructure. Child nodes are scattered where there is no power and no network, so they run autonomously on their chosen local power source. Parents are few, so instead of making them self-powered too, we place them where power already exists: pump stations, utility poles, vending machines, community halls. Choosing a resilient host (e.g. a pump station with backup power) also buys the parent's blackout tolerance for free. Deployment note: using such infrastructure requires permission from its manager (road authority, utility, water/sewer division).

8. Optimize for your land — choose a power source that matches your climate and constraints. Power is deliberately left as an open design choice, because the right answer depends on your sunlight, terrain, budget, and maintenance capacity. Our own prototype uses a small solar panel with a supercapacitor (EDLC) — no chemical rechargeable batteries, no leakage or fire risk, very long cycle life, tolerant of heat and cold — because it suits a sunny, low-rainfall-variance region (developed in Okayama, a part of Japan known as the "Land of Sunshine"). But this is only one option among several:

Solar + supercapacitor (EDLC) — maintenance-free where sunlight is reliable; our default.
Primary (non-rechargeable) batteries, e.g. lithium CR cells — extremely simple, excellent shelf life and cold-weather performance, and viable precisely because our duty cycle is tiny (the node sleeps almost all the time and only sends a short heartbeat every 2 hours). In shaded gutters, dense-canopy areas, or long-overcast monsoon climates where solar is unreliable, a primary cell may outlast and out-simplify a solar rig. The tradeoff is an eventual (but rare and predictable) replacement.
Solar + rechargeable battery (LiFePO₄ etc.) — more stored energy for long dark spells, at the cost of finite cycle life and thermal considerations.
Hybrid (solar + supercap + small primary cell as a fallback) — the supercap handles daily cycling; the primary cell carries the node through extended dark periods.
The key point is that the extremely low duty cycle of this design makes almost any of these viable. Because the node does nothing 99.9% of the time, it does not demand a large or expensive power source. This is also why our headline goal is maintenance-minimized rather than strictly "battery-free": with solar + supercap a node can run untouched for years, and even with a primary cell, replacement is rare and predictable. This open source exists so you can pick — and retune — the power design for your own land.

9. The microcontroller is disposable; the standard is the asset. By building on the industry-standard Thread / IEEE 802.15.4, a better chip in the future can replace the nodes while the network stays intact. Open source and open standards protect this "freedom to migrate."

Architecture
Copy[ Each gutter / underpass / school route — child node ]
  Power source (choose to suit your site):
    Solar + supercapacitor (EDLC) / primary cell (e.g. CR) / solar + rechargeable / hybrid
        |
        v
  ESP32-C6 (Thread) + electrode water detection (5 cm / 15 cm)
        |
        v  autonomous mesh relay, ~100-200 m spacing
[ node ] -> [ node ] -> ... -> [ Parent = Border Router on existing infra ]
        |
        v  existing Wi-Fi  (or LTE-M where no Wi-Fi)
  Google Apps Script (backend + API)
        |
   +----+---------------------------+
   v                                v
 Dashboard (4-color map)     Public messaging channel (alerts)
Status colors: blue (normal) / yellow (5 cm) / red (15 cm) / black (no response = LOST = highest danger).

Who might find this useful
We believe this design is most valuable not where money is abundant, but where it is scarce — regions facing recurring floods without the budget to sustain expensive monitoring. Underpasses, where vehicles drown every year, are a particularly strong use case. If you are a researcher, an NGO, a teacher, or a community maker working on low-cost flood early warning, please take this, break it, and rebuild it for your own conditions. That is what it is for.

Relationship to existing efforts
Japan's national government (MLIT) runs a large-scale "One-Coin Flood Sensor" field trial. This project does not compete with it and is not a replacement. It is a way to ask: "The nation answered this challenge one way — how would we answer it?" Public programs and grassroots ingenuity differ in speed, coverage, and accountability, and understanding that difference is part of the lesson.

Status
An educational prototype under active development. Expect rough edges. Contributions, forks, and regional adaptations are warmly welcomed.

Disclaimer
Provided for educational purposes. No guarantee is made as to accuracy, availability, or fitness for any purpose. The authors accept no liability for any damages arising from use of this system, including its use in evacuation decisions. Always follow official information from your local authorities and meteorological agency.

License
All materials in this repository — software, firmware, circuit schematics, and educational curricula, manuals, and documentation — are released under the MIT License. See LICENSE.

日本語
このプロジェクトの目的
洪水は毎年、水を監視する余裕のない地域で命を奪っています。市販の浸水監視システムは、密に設置するには高価すぎ、そして何より「動かし続ける」コストが高すぎます。大規模なセンシングの本当の敵は、センサ本体の値段ではありません。通信費・電池交換・サーバ費用・終わりのない保守——毎年かかり続けるランニングコストです。

このプロジェクトは、私たちが、たったひとつの問いに答えようとした試みです。

めったに起きない災害を、どれだけ安く、頑丈に、少ない手間で「待ち構え続けられるか」。

なぜなら浸水監視とは、まさにそれ——何も起きない99.9%の時間を、いかに備えたまま過ごせるか、の戦いだからです。

設計思想
1. 「何も検知しないこと」が正常である。 このシステムは大半の時間、何も検知しません。そしてそれが正しい状態です。難しいのは水を感じることではなく、それを安く、無期限に、できるだけ手間をかけずに「待つ」ことです。

2. 足し算ではなく引き算 ― 守らないものを決める。 すべてを守ろうとすると、高価で壊れやすくなります。水位は連続値では測らず、二本の電極の高さの差で 5cm(注意) と 15cm(危険) の二段階だけを検知します。それを超える冠水では、デバイス自身が水没して応答を止めます。私たちはこの**「応答なし(ロスト)」という状態そのものを、最大の危険信号(黒)として扱います。** センサが黙ることが、最後の、そして最も雄弁な通信になります。

3. フェイルセーフ設計 ― 沈黙を警報とする(B接点思考)。 多くの浸水センサは「水が来たら知らせる」設計(A接点/常開)で、平常時は沈黙します。しかしこの沈黙は「安全」と「故障」を区別できません。本システムはこれを逆転させます。平常時こそ生存信号(ハートビート)を送り続け(B接点/常閉の発想)、その途絶——沈黙そのもの——を最大の警報として扱います。 デバイスが水没して機能を止めても、その死そのものが「ここは危険水位を超えた」という情報になります。だからこそ本機は水没して構わない設計であり(IP56相当で十分)、防水に過剰なコストをかけません。壊れたら危険側ではなく安全側=「警報側」に倒れる——鉄道信号やエレベーター安全回路と同じフェイルセーフ思想です。私たちが作りたくないのはその逆、すなわち「静かに壊れ、次の洪水まで誰も気づかないセンサ」です。

4. 普段は寝て、水が来たら起きる。 電極は平常時には通電せず、消費電力はほぼゼロです。マイコンは 2時間おきにハートビート(生存確認) だけを送り、また眠ります。この間隔は、電力の節約・ロスト検知の速さ・通信の混雑という三つのバランスで決めました。この極端に低いデューティ比こそが、必要な電力を極小に保ち、結果として「ほぼどんな電源方式でも成立する」ことを可能にしています(→設計思想8)。

5. 二段構えの監視モード。 5cm に達した子機は、自ら観測間隔を約10分に切り替えます(自律格上げ)。さらに、大雨の予報などに応じて、中央のサーバから 全子機へ「観測頻度を上げよ」という指令を一斉に下ろすこともできます(中央格上げ)。正直な限界: 省電力のため子機が眠っている間は、指令の反映に最大でハートビート間隔ぶんの遅れが生じます。この「即応性と省電力のトレードオフ」は、私たちが今も向き合っている課題です。

6. 通信のランニングコストを「構造」で消す。 1台ごとに毎年課金される携帯回線(LPWA / LTE)を使わず、子機どうしが Thread(IPv6ベースの自己修復メッシュ) で中継し合い、親機(Border Router)が既存インフラへ橋渡しします。多数の子機を1台の親機で束ねるため、外部回線が必要なのは親機だけ——これで運用の維持費を構造的にほぼゼロにします。親機は、利用できる場所では 既存Wi-Fi(学校・公民館)を、無い場所では LTE-M を使います。1台が失われても、周囲の子機が別の中継ルートを自動で組み直します。

7. 親機は既存インフラに相乗りさせる。 子機は電気もネットもない場所に散らばるため、その土地に合わせて選んだ電源で自律動作します。一方、親機は数が少ないので、これも自律させるのではなく、すでに電気が来ている場所——ポンプ場、電柱、自販機、公民館——に相乗りさせます。 非常用電源を備えた拠点(例:ポンプ場)を選べば、親機の停電耐性まで拠点側から借りられます。設置上の注意: こうしたインフラの利用には、その管理者(道路管理者・電力会社・上下水道部局等)の許可が必要です。

8. 土地に合わせて最適化する ― 気候と制約に合った電源を「選ぶ」。 電源は、あえて開かれた設計選択として残しています。最適解は、その土地の日照・地形・予算・保守体制によって変わるからです。私たち自身の試作機は、小型の太陽光パネルとスーパーキャパシタ(EDLC)を採用しています。充電式の化学電池を使わないため液漏れ・発火のリスクがなく、超高サイクル寿命で高温・低温にも強く、日照が多く天候の振れが小さい地域(開発地は「晴れの国」岡山)に適しているからです。しかしこれは、いくつかある選択肢の一つにすぎません。

ソーラー+スーパーキャパシタ(EDLC) ―― 日照が安定した土地でメンテナンスフリー。私たちの標準構成です。
一次電池(交換式・非充電/例:リチウムCR電池) ―― きわめてシンプルで、長期保存性と低温性能に優れます。そして本システムのデューティ比が極端に小さいからこそ現実的な選択肢です(子機はほぼ常に眠っており、2時間に一度、短いハートビートを送るだけ)。日陰の側溝、樹冠の濃い場所、連日曇天が続くモンスーン気候など、ソーラーが当てにならない環境では、一次電池のほうが長持ちし、構成もシンプルになり得ます。引き換えは、まれではあるが予測可能な「いつかの電池交換」です。
ソーラー+充電式電池(LiFePO₄ 等) ―― 長い曇天・夜間に備えて蓄電量を増やせます。引き換えにサイクル寿命の限界と温度への配慮が必要です。
ハイブリッド(ソーラー+スーパーキャパシタ+予備の一次電池) ―― 日々の充放電はスーパーキャパシタが担い、長い日照不足の期間を一次電池が下支えします。
肝心なのは、本設計の極端に低いデューティ比が、これらのほぼどの方式も成立させるという点です。 子機は99.9%の時間、何もしていません。だからこそ、大容量で高価な電源を必要としません。私たちが掲げる目標を厳密な「電池交換フリー」ではなく「保守の手間の最小化」としているのもこのためです。ソーラー+スーパーキャパシタなら子機は何年も無交換で動き、一次電池でも交換はまれかつ予測可能です。このオープンソースは、あなたが自分の土地に合わせて電源方式を「選び」、調整し直せるように存在します。

9. マイコンは消耗品、規格は資産。 業界標準の Thread / IEEE 802.15.4 の上に築いたことで、将来より優れたチップが出れば、ネットワークはそのままに子機だけを載せ替えられます。オープンソースと標準規格は、この「未来への乗り換え可能性」を守るためにあります。

システム構成
Copy[ 各側溝・アンダーパス・通学路 — 子機 ]
  電源(設置場所に合わせて選択):
    ソーラー+スーパーキャパシタ(EDLC) / 一次電池(例:CR) / ソーラー+充電式 / ハイブリッド
        |
        v
  ESP32-C6 (Thread) + 電極による水位検知 (5cm / 15cm)
        |
        v  100〜200m間隔の自律メッシュリレー
[ 子機 ] -> [ 子機 ] -> ... -> [ 親機 = 既存インフラ上の Border Router ]
        |
        v  既存Wi-Fi (無ければ LTE-M)
  Google Apps Script (バックエンド + API)
        |
   +----+---------------------------+
   v                                v
 ダッシュボード(4色マップ)     住民向け通知チャンネル(アラート)
状態の色: 青(平常) / 黄(5cm) / 赤(15cm) / 黒(応答なし = ロスト = 最大の危険)。

誰の役に立つか
この設計が最も価値を持つのは、お金が潤沢な場所ではなく、乏しい場所——高価な監視を維持する予算がないまま、毎年洪水に直面する地域だと私たちは考えています。毎年車両の水没事故が起きるアンダーパスは、特に有力な用途です。低コストの洪水早期警報に取り組む研究者、NGO、教員、地域の作り手の方は、どうかこれを持っていき、壊し、自分の土地の条件に合わせて作り直してください。そのためのオープンソースです。

既存の取り組みとの関係
日本政府(国土交通省)は「ワンコイン浸水センサ」の大規模な実証実験を進めています。本プロジェクトはそれと競うものでも、代わるものでもありません。「同じ課題に、国はこう答えた。では私たちはどう答えられるか」 を考えるための取り組みです。公的な取り組みと草の根の工夫は、速さ・網羅性・責任の所在という点で役割が違う——それを理解することも学びの一部です。

状態
開発中の教育用プロトタイプです。粗削りな点があります。貢献・フォーク・各地域向けの改変を歓迎します。

免責事項
本成果物は教育目的で提供されます。正確性・可用性・特定目的への適合性を保証しません。作者は、本システムの使用(避難判断への利用を含む)から生じるいかなる損害についても責任を負いません。避難に関する判断は、必ず自治体および気象庁の公式情報に従ってください。

ライセンス
本リポジトリに含まれるすべての成果物(ソフトウェア・ファームウェア・回路図・教育用カリキュラム・マニュアル・文書)を MIT ライセンスのもとで公開します。詳細は LICENSE を参照してください。
