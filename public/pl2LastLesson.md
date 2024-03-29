---
title: 情報学群実験第2最終課題設計書
tags:
  - RaspberryPi
  - assembly
  - ARM
private: true
updated_at: '2024-01-12T14:03:09+09:00'
id: 24b78743f032755cedef
organization_url_name: null
slide: false
ignorePublish: false
---

# 情報学群実験第2最終課題設計書

<details><summary>参考</summary>

[PWMで和音](https://ameblo.jp/ringworldengineer/entry-12345071870.html)

</details>

## 概要
このプロジェクトは、8x8マスのスネークゲームをアセンブリ言語で実装するものである。<br>
ゲームは砲弾型LED(緑)、ビープスピーカー、および4つの操作用ボタンを使用してプレイヤーが操作する。
<details><summary>盤面配置</summary>

|       |       |       |       |       |       |       |       |          |       |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :------: | :---: |
|  緑   |       |       |       |       |       |       |       | スピーカ |       |
| 赤00  | 赤01  | 赤02  | 赤03  | 赤04  | 赤05  | 赤06  | 赤07  |          |       |
| 赤10  | 赤11  | 赤12  | 赤13  | 赤14  | 赤15  | 赤16  | 赤17  |          |       |
| 赤20  | 赤21  | 赤22  | 赤23  | 赤24  | 赤25  | 赤26  | 赤27  |          |       |
| 赤30  | 赤31  | 赤32  | 赤33  | 赤34  | 赤35  | 赤36  | 赤37  |          |       |
| 赤40  | 赤41  | 赤42  | 赤43  | 赤44  | 赤45  | 赤46  | 赤47  |          |       |
| 赤50  | 赤51  | 赤52  | 赤53  | 赤54  | 赤55  | 赤56  | 赤57  |          |       |
| 赤60  | 赤61  | 赤62  | 赤63  | 赤64  | 赤65  | 赤66  | 赤67  |  青SW3   | 赤SW4 |
| 赤70  | 赤71  | 赤72  | 赤73  | 赤74  | 赤75  | 赤76  | 赤77  |  黒SW1   | 青SW2 |
</details>

## 機能要件
* スネークの成長と移動
* 4つのボタンを使用した方向制御
* マス上に表示されるLEDによるゲーム状態表示
* 単音式スピーカーを使用したゲームイベントのサウンド通知
* 

## ハードウェア構成
- 8x8 LEDマトリックス（赤）
- 4つの方向ボタン (上、下、左、右)
- LED（ゲーム状態表示）（緑）
- 単音式スピーカー

## プログラム構造
* ランダムな位置にりんご
* スイッチで上下左右に移動
* りんごを食べたら蛇が一マス伸びる
* 蛇がすべてのマスを埋めたらゲームクリア
* 蛇が壁または蛇自身にあたったらゲームオーバ

### 詳細
* 蛇の移動速度は1.5sづつ
* 蛇は常時点灯
* (できたら)蛇の頭から明るさを1/5づつにする
* 蛇の初期動作方向は右
* りんごは0.4,0.1ペースでチカチカ
* りんごはステージに一つ 食べられたら出現
* りんごを食べたらLEDが2秒で4回チカチカ
* クリアした場合LEDはずっと点灯
* ゲームオーバーの場合LEDはチカチカ
  * また画面に `×` をチカチカ表示

### ゲームループ
1. settingの呼び出し
2. 初期画面の表示 (例えば `00` を表示)
3. 音楽の再生開始
4. (1,1)にスネークの頭、(1,0)スネークの尾を表示
5. スネークがいる場所以外にりんごを表示
6. ボタン入力の取得
7. 入力に基づいてスネークを移動
8. スネークが食べ物を食べた場合、成長
9. スネークが壁または蛇自身に当たった場合、ゲームオーバー
10. ゲーム状態のLED表示の更新

## コード構成例
* メイン: `.section .init`のある実行開始ルーチン
### 以下サブルーチン
* setting.s: 初期設定をまとめる (イニシャライザ)
* display.s: frame_bufferを表示
  * frame_bufferを内蔵
  * input:  none
  * output: none
* number.s: frame_bufferに2桁の数字を表示
  * input: r1: 数字, r2: 進数
  * output: none
* switch1.s: 押されたボタンを返す
* switch2.s: 〃
* switch3.s: 〃
* switch4.s: 〃
  * input:  none
  * output: r0: 0-押してない 1-押している
* ledon.s: 砲弾型LEDを点灯させる
  * input:  none
  * output: none
* ledoff.s: 砲弾型LEDを消灯させる
* random0to63.s: 0から63までのランダムな値を返す
  * input: none
  * output: r0: 0 to 63 のランダムな値
* sound.s: 一音鳴らす
  * input: r1: 音のコード (69 A4 440)
  * output: none


and more…

## サブルーチン化
[--- LMS資料 ---](https://lms.kochi-tech.ac.jp/pluginfile.php/207510/mod_resource/content/1/%E3%82%B5%E3%83%95%E3%82%99%E3%83%AB%E3%83%BC%E3%83%81%E3%83%B3.pdf)

プログラム冒頭に仕様を記述したコメントを書くこと
### コード例
```assembly:random0to63.s
@--------------------------------------------------
    @ random0to63
    @     xorshiftにより0から63までのランダムな値を返す
    @ input:
    @     none
    @ return_value:
    @     r0: 0 to 63 のランダムな値
@--------------------------------------------------
    .section .text
    .global random0to63
random0to63:
    push    {r1 - r3, lr}

    @ 初期値
    add     r0, r0, r1
    add     r0, r0, r2
    add     r0, r0, r3
    add     r0, r0, r4
    add     r0, r0, r5
    add     r0, r0, r6
    add     r0, r0, r7
    add     r0, r0, r8
    add     r0, r0, r9
    add     r0, r0, r10
    add     r0, r0, r11
    add     r0, r0, r12
    add     r0, r0, r13
    add     r0, r0, r14
    add     r0, r0, r15

    @ x ^= x << 3;
    mov     r1, r0, lsl #3
    eor     r0, r1, r0

    @ x ^= x >> 13;
    mov     r1, r0, lsr #13
    eor     r0, r1, r0

    @ x ^= x << 7;
    mov     r1, r0, lsl #7
    eor     r0, r1, r0

    @ abs(x);
    movs    r1, r0
    rsbmi   r1, r1, #0

    @ x % 64
    mov     r2, #64
    udiv    r3, r1, r2
    mul     r0, r3, r2
    sub     r0, r1, r0

    pop     {r1 - r3, lr}
    bx      lr
```
