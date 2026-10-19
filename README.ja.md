# CAN_DBC_MAZDA_MX-5 2015-1.5L/MT/JP

日本仕様の2015年式マツダ ロードスター（MX-5／ND、1.5L・6MT）で収集したCANデータを基に整理したDBCです。

出発点は、**berumiya 990S labo氏の[CAN_DBC_6thGenMazda](https://github.com/berumiya/CAN_DBC_6thGenMazda)**です。元DBCで動作させた後、自車のCANログと実際の操作を照合し、Codexを解析補助として年式差の検討や定義の追加・修正を進めました。元DBCと解析結果を公開してくださったberumiya氏に感謝します。

元プロジェクトの主な検証車両は2022年式990Sです。本リポジトリでは2015年式の自車で得られた情報をまとめています。

## 対象とファイル

| 項目 | 内容 |
|---|---|
| 車両 | マツダ ロードスター／MX-5、2015年式 S、ND5RC |
| エンジン・変速機 | 1.5L、6速MT |
| 仕向地 | 日本仕様（JP） |
| CAN | Classic CAN、500 kbit/s |
| ファイル | [MAZDA_MX-5_2015_1.5L_MT_JP.dbc](MAZDA_MX-5_2015_1.5L_MT_JP.dbc) |
| 内容 | 75メッセージ、176信号 |
| 元となる版 | ND1-2015-MT-20261019-R2、v1.4.1 |

## 使い方

1. 上のDBCファイルを開き、**Download raw file**から保存してください。リポジトリ全体を取得する場合は、**Code → Download ZIP**を使えます。
2. DBC対応のCAN解析ツールへ読み込み、CANインターフェースの通信速度を500 kbit/sに設定してください。実車からの収集には**Listen Only（受信専用）**を推奨します。
3. 信号の単位・備考・列挙値を確認して利用してください。同じビットを異なる候補として読む信号を含むため、読込み時に重複チェックの調整が必要なツールがあります。

例えば、cantoolsが使えるPython環境では次のように読み込めます。

```python
import cantools

db = cantools.database.load_file(
    "MAZDA_MX-5_2015_1.5L_MT_JP.dbc",
    strict=False,
)
```

## 元DBCからの主な変更

- 2015年式でのギア・燃料・灯火などの読み方を整理。
- 後退選択、後退灯、ブレーキ灯、クラッチ状態の候補を追加。
- ブレーキ圧力候補を20bitとして定義。
- 外気温を`0x420 byte7 × 0.25 ℃`として定義。
- 未解読メッセージのpayloadをraw値として保持。

特殊な書式のVIN関連3信号は、この標準DBCには含めていません。

## 信号を読む際の補足

byte番号は0始まりです。DBCのMotorola形式の開始bit番号と、byte内のbit番号は異なる表記です。

| 信号 | 読み方・補足 |
|---|---|
| `Fuel_Tank`（0x09E） | byte5 × 0.2 Lの燃料計値。傾斜や加減速で変動するため、安定した停車時の値を残量の基準にします。 |
| `Fuel_Related`（0x43F）／`Fuel_Related_2`（0x09E） | 意味不明のraw値。Lや％への換算、補正済み残量としての扱いは加えていません。 |
| `C000_TOTAL_TIME`（0x40A） | `raw × 0.1 + 1 sec`の秒カウンター。起点・用途は不明です。 |
| `BrakePressure`（0x078） | Motorola `27\|20@0+`、`raw × 0.001 − 40 bar`の圧力候補。bar換算と下位byteの解釈は暫定です。 |
| `SteeringAngle_related`（0x086） | 左操舵で増加するraw値。参考式`(raw − 16000) × 0.1 deg`のゼロ点・倍率は暫定で、DBC自体はrawのままです。 |
| `Steering_Wheel_Absolute_Angle`（0x082） | 原本の定義を保持していますが、この車両の舵角観測とは一致していません。 |
| `WheelSpeed_1`〜`WheelSpeed_4`（0x215） | km/h単位の車輪速。1／3は左、2／4は右の対応で、前後の割当は不明です。 |
| `Travel_distance_related`（0x420） | raw 0〜254、modulo 255の距離カウンター。約0.2 m/countの整合がありますが、DBCの単位・倍率はrawのままです。 |
| `AmbientTemp`（0x420） | byte7 × 0.25 ℃。負温度の符号化や無効値は不明です。 |
| `Clutch_State_Candidate`（0x166） | raw 0／62／187の3状態を解放側／中間／深い踏込み側の候補として扱います。踏力や移動量の％ではありません。 |
| `MT_Gear_Actual`（0x0FD） | 1〜6速。変速時には更新が遅れることがあります。後退候補は別信号です。 |

G・ヨーレートにはraw系と補正系の候補があり、物理量への倍率や両者の関係に不明点が残ります。DBC内のmin/maxには原本由来の仮値も含まれるため、実際のセンサー範囲を示す値としては扱わないでください。

### 0x40Aの読み分け

0x40Aは、先頭2バイトの識別値（`Central_Config_Index`）に応じて、後続データの読み方が切り替わるメッセージです。現在は次のように扱っています。

| 識別値 | 信号 | 読み方 |
|---|---|---|
| `0xC000` | `C000_TOTAL_TIME` | `raw × 0.1 + 1 sec`の秒カウンター。起点・用途は不明。 |
| `0xC001` | `C001_ODO` | raw値をkm単位の走行距離計として読む。 |
| `0xBF00` | `BF00_IG_ON_Timer` | raw値をエンジン始動後の秒数として読む。リセット条件は不明。 |

その他の識別値の内容はraw値として保持しています。

## ブレーキの参考尺度と参照元

ブレーキの％換算の検討には、Timur Iskhodzhanov氏の[RaceChronoDiyBleDeviceのND用資料](https://github.com/timurrrr/RaceChronoDiyBleDevice/blob/841c7ab958e92b83953398475e4fc7cdd1c64771/can_db/mazda_mx5_nd.md)を参照しました。

同資料は、0x078の12bit値から156を引いて2.56で割り、0〜100％へ制限する式を示しています。自車で検討した相対尺度は、20bit候補の上位12bitを`q = raw >> 8`として、`max(q − 156, 0) × 100 / 256`です。100％を超える値も計算できます。

この100％は参考尺度上の基準で、実踏力・ペダル移動量・油圧限界を校正した最大値ではありません。**DBCの`BrakePressure`はbarの圧力候補を定義し、この％換算は含みません。** 20bitの圧力候補と自車での照合は本プロジェクトの検討内容であり、参照元の校正結果を示すものではありません。

## 使用上の注意・免責事項

本 DBC の内容はメーカー公式資料ではなく、個人による解析に基づいたものです。内容の正確性・完全性を保証するものではありません。

本データを使用したことによるいかなる損害についても、作者は責任を負いません。

作者は車両へのCANメッセージ送信や、制御用途での使用は一切検証しておりません。

## 出典とライセンス

- 元DBC：**berumiya 990S labo**、[CAN_DBC_6thGenMazda](https://github.com/berumiya/CAN_DBC_6thGenMazda)、commit `c802353b4e52546e196b8ee5dc483f73c14dbc2b`。
- 自車での追加解析・修正・配布用整理：**takakka**。解析補助にCodexを使用。
- 参考資料：**Timur Iskhodzhanov**、[RaceChronoDiyBleDevice](https://github.com/timurrrr/RaceChronoDiyBleDevice)、commit `841c7ab958e92b83953398475e4fc7cdd1c64771`、`can_db/mazda_mx5_nd.md`。

DBCと本資料の追加・修正分は[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)で提供します。[LICENSE](LICENSE)と[THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md)を参照してください。元の帰属表示を保持し、再配布時は出典・ライセンス・変更した旨を示してください。

RaceChronoDiyBleDeviceの参照情報には、同プロジェクトの[MITライセンス](licenses/RaceChronoDiyBleDevice-MIT.txt)の通知を保持しています。元作者や車両メーカーによる承認・監修を示すものではありません。
