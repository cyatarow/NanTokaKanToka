# Keychron C3 Pro 8K（ANSI配列）の自分用QMKファームウェア改造
元レポジトリ→ https://github.com/Keychron/qmk_firmware  
同じC3 Pro 8KでもJIS配列ユーザーの場合、「keyboards/keychron/c3_pro_8k/ansi/」フォルダを「keyboards/keychron/c3_pro_8k/jis/」に読み替え、keymap.cについてもJIS用に修正する必要あり

----

## keyboards/keychron/common/rgb/keychron_rgb_type.h
- 20行目からのenum 1箇所：誤字修正（プルリクエスト https://github.com/Keychron/qmk_firmware/pull/472/changes より）

```
enum {
    PER_KEY_RGB_SOLID,
    PER_KEY_RGB_BREATHING,
    PER_KEY_RGB_REACTIVE_SIMPLE,
    PER_KEY_RGB_REACTIVE_MULTI_WIDE,
    PER_KEY_RGB_REACTIVE_SPLASH,
    PER_KEY_RGB_MAX,
};
```

----

## keyboards/keychron/common/rgb/keychron_rgb.c
- 248行目：条件修正（プルリクエスト https://github.com/Keychron/qmk_firmware/pull/472/changes より）

`if (count > 3 || region >= EFFECT_LAYERS || start + count > EFFECTS_PER_LAYER) return false;`

- 266行目：条件修正

`if (count > 3 || region >= EFFECT_LAYERS || start + count > EFFECTS_PER_LAYER) return false;`

- 436行目：条件修正

`if (host_keyboard_led_state().scroll_lock && !os_ind_cfg.disable.scroll_lock) {`

----

## keyboards/keychron/common/rgb/per_key_rgb.c
- 35行目：キーごとのRGB LEDの色指定にて、色の明度（HSV変換後の「V」の値）が強制的にキーボード全体の明度（Keychron Launcherにて1～10の10段階で設定できる）に合わせられるという、不親切な挙動を修正  
具体的には、《#800000と#FF0000のどちらを指定しても、同じ明度の赤点灯になる》《#000000と指定すると黒（消灯）ではなく白点灯になる》といった点である  
（プルリクエスト https://github.com/Keychron/qmk_firmware/pull/472/changes に寄せられたコメントをもとに再構成した）

`hsv.v = scale8(hsv.v, rgb_matrix_config.hsv.v); // <元の挙動> hsv.v = rgb_matrix_config.hsv.v;`

- 42行目：誤字修正

`bool per_key_rgb_breathing(effect_params_t *params) {`

- 49行目：35行目と同様の修正を「ブリージング」効果指定時にも適用

`hsv.v = scale8(abs8(sin8(time) - 128) * 2, scale8(hsv.v, rgb_matrix_config.hsv.v)); // <元の挙動> hsv.v = scale8(abs8(sin8(time) - 128) * 2, rgb_matrix_config.hsv.v);`

- 77行目：35行目と同様の修正を「シングルキー・レスポンス」効果指定時にも適用

`hsv.v = scale8(255 - offset, scale8(hsv.v, rgb_matrix_config.hsv.v)); // <元の挙動> hsv.v = scale8(255 - offset, rgb_matrix_config.hsv.v);`

- 107行目：35行目と同様の修正を「隣接キー・レスポンス」効果指定時にも適用

`hsv.v = scale8(hsv.v, scale8(per_key_led[i].v, rgb_matrix_config.hsv.v)); // <元の挙動> hsv.v = scale8(hsv.v, rgb_matrix_config.hsv.v);`

- 145行目：誤字修正

`return per_key_rgb_breathing(params);`

- 147行目：誤字修正

`case PER_KEY_RGB_REACTIVE_SIMPLE:`

- 150行目：誤字修正

`case PER_KEY_RGB_REACTIVE_MULTI_WIDE:`

- 153行目：誤字修正

`case PER_KEY_RGB_REACTIVE_SPLASH:`

----

## keyboards/keychron/c3_pro_8k/ansi/ansi.c
- 136行目のDC_RED定義から、HSV default_per_key_led定義まで：私の好みで、キーごとのRGB LED初期設定を変更  
特に、CapsLockとInsertにあたる部分を消灯している

```
#define DC_RED {HSV_RED}
#define DC_BLU {HSV_BLUE}
#define DC_YLW {HSV_YELLOW}

// 自分用:
#define AREA_0 {  0,   0,   0} //Black
#define AREA_1 {  0, 255, 255} //Red
#define AREA_2 { 37, 255, 255} //Yellow
#define AREA_3 { 85, 255, 255} //Green
#define AREA_4 { 98, 255, 255} //Blue-Green
#define AREA_5 {159, 255, 255} //Blue
#define AREA_6 {183, 255, 255} //Blue-Purple

HSV default_per_key_led[RGB_MATRIX_LED_COUNT] = {
    AREA_1, AREA_1, AREA_1, AREA_1, AREA_1, AREA_1, AREA_1, AREA_1, AREA_1, AREA_1, AREA_1, AREA_1, AREA_1,           AREA_6, AREA_6, AREA_6,
    AREA_4, AREA_4, AREA_4, AREA_4, AREA_4, AREA_4, AREA_4, AREA_4, AREA_4, AREA_4, AREA_4, AREA_4, AREA_4, AREA_2,   AREA_0, AREA_6, AREA_6,
    AREA_2, AREA_3, AREA_3, AREA_3, AREA_3, AREA_3, AREA_3, AREA_3, AREA_3, AREA_3, AREA_3, AREA_4, AREA_4, AREA_4,   AREA_6, AREA_6, AREA_6,
    AREA_0, AREA_3, AREA_3, AREA_3, AREA_3, AREA_3, AREA_3, AREA_3, AREA_3, AREA_3, AREA_4, AREA_4,         AREA_2,
    AREA_2,         AREA_3, AREA_3, AREA_3, AREA_3, AREA_3, AREA_3, AREA_3, AREA_4, AREA_4, AREA_4,         AREA_2,           AREA_5,
    AREA_2, AREA_2, AREA_2,                         AREA_3,                         AREA_2, AREA_2, AREA_2, AREA_2,   AREA_5, AREA_5, AREA_5,
    // Left side leds
    AREA_0, AREA_0, AREA_0, AREA_0, AREA_0, AREA_0,
    // Right side leds
    AREA_0, AREA_0, AREA_0, AREA_0, AREA_0, AREA_0,
};
```

----

## keyboards/keychron/c3_pro_8k/ansi/config.h
- 29行目：LED電流を増やして、もっと明るく

`        { 0x70, 0x70, 0x70, 0x70, 0x70, 0x70, 0x70, 0x70, 0x70, 0x70, 0x70, 0x70 } // <元の挙動> { 0x28, 0x28, 0x28, 0x28, 0x28, 0x28, 0x28, 0x28, 0x28, 0x28, 0x28, 0x28 }`

- 最下部に追加：Bootmagicを“実質的に”無効化  
特定のキーを押しながらUSB接続するとブートローダーに入れる機能「Bootmagic」  
デフォルトでは0行0列目のキーに割り当てられ、テンキーレスキーボードなら通例Escキーを指す  
しかし誤操作のおそれが拭いきれないので、Bootmagicを無効化したかった  
そこでrules.mkで「BOOTMAGIC_ENABLE = no」を指定したものの、無効にならなかった  
原因を調べると、どうやらVIAが有効（VIA_ENABLE = yes）だとBootmagicも強制的に有効になることが判明  
VIAを切るとKeychron Launcherも動作しなくなるため、VIAを残したままBootmagicを実質無力化する方法を探した結果、  
このconfig.hに定数「BOOTMAGIC_ROW」と「BOOTMAGIC_COLUMN」を定義し、“キーが物理的に存在しない位置”を指定することで、実質無力化に成功した  
0行0列目にEscキーが、0行2列目にF1キーが存在しているが、間の「0行1列目」にキーの定義も物理的存在もないことを利用した（keyboard.jsonを参照）

```
#define BOOTMAGIC_ROW 0 	
#define BOOTMAGIC_COLUMN 1
```

----

## keyboards/keychron/c3_pro_8k/ansi/keymaps/MY_MAPPING/
- 自分用キーマップフォルダとして新規作成

----

## keyboards/keychron/c3_pro_8k/ansi/keymaps/MY_MAPPING/keymap.c
- 新規作成：Windows用キーマップの初期設定を、自分好みに改造  
CapsLockとInsertを無効化し、Fn・メニュー・ScrollLock・Pauseの4キーを通常のANSIキーボードと同じ位置に再配置する  
さらに[WIN_FN]レイヤーにはQK_BOOTキーコードを新設する（上記でBootmagicを実質無効化した代わり）

```
/* Copyright 2025 @ Keychron (https://www.keychron.com)
 *
 * This program is free software: you can redistribute it and/or modify
 * it under the terms of the GNU General Public License as published by
 * the Free Software Foundation, either version 2 of the License, or
 * (at your option) any later version.
 *
 * This program is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
 * GNU General Public License for more details.
 *
 * You should have received a copy of the GNU General Public License
 * along with this program. If not, see <http://www.gnu.org/licenses/>.
 */

#include QMK_KEYBOARD_H
#include "keychron_common.h"

enum layers {
    MAC_BASE,
    MAC_FN,
    WIN_BASE,
    WIN_FN,
};

#define FN_MAC MO(MAC_FN)
#define FN_WIN MO(WIN_FN)

// clang-format off
const uint16_t PROGMEM keymaps[][MATRIX_ROWS][MATRIX_COLS] = {
    [MAC_BASE] = LAYOUT_tkl_ansi(
        KC_ESC,             KC_BRID,  KC_BRIU,  KC_MCTRL, KC_LNPAD, UG_VALD,  UG_VALU,  KC_MPRV,  KC_MPLY,  KC_MNXT,  KC_MUTE,   KC_VOLD, KC_VOLU,  KC_SNAP,  KC_SIRI,  UG_NEXT,
        KC_GRV,   KC_1,     KC_2,     KC_3,     KC_4,     KC_5,     KC_6,     KC_7,     KC_8,     KC_9,     KC_0,     KC_MINS,   KC_EQL,  KC_BSPC,  KC_INS,   KC_HOME,  KC_PGUP,
        KC_TAB,   KC_Q,     KC_W,     KC_E,     KC_R,     KC_T,     KC_Y,     KC_U,     KC_I,     KC_O,     KC_P,     KC_LBRC,   KC_RBRC, KC_BSLS,  KC_DEL,   KC_END,   KC_PGDN,
        KC_CAPS,  KC_A,     KC_S,     KC_D,     KC_F,     KC_G,     KC_H,     KC_J,     KC_K,     KC_L,     KC_SCLN,  KC_QUOT,            KC_ENT,
        KC_LSFT,            KC_Z,     KC_X,     KC_C,     KC_V,     KC_B,     KC_N,     KC_M,     KC_COMM,  KC_DOT,   KC_SLSH,            KC_RSFT,            KC_UP,
        KC_LCTL,  KC_LOPTN, KC_LCMMD,                               KC_SPC,                                 KC_RCMMD, KC_ROPTN,  FN_MAC,  KC_RCTL,KC_LEFT,    KC_DOWN,  KC_RGHT),

    [MAC_FN] = LAYOUT_tkl_ansi(
        _______,            KC_F1,    KC_F2,    KC_F3,    KC_F4,    KC_F5,    KC_F6,    KC_F7,    KC_F8,    KC_F9,    KC_F10,   KC_F11,   KC_F12,   _______,  _______,  UG_TOGG,
        _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,
        UG_TOGG,  UG_NEXT,  UG_VALU,  UG_HUEU,  UG_SATU,  UG_SPDU,  _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,
        OS_TOGGL, UG_PREV,  UG_VALD,  UG_HUED,  UG_SATD,  UG_SPDD,  _______,  _______,  _______,  _______,  _______,  _______,            _______,
        _______,            _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,            _______,            _______,
        _______,  _______,  _______,                                _______,                                _______,  _______,  _______,  _______,  _______,  _______,  _______),

    [WIN_BASE] = LAYOUT_tkl_ansi(
        KC_ESC,             KC_F1,    KC_F2,    KC_F3,    KC_F4,    KC_F5,    KC_F6,    KC_F7,    KC_F8,    KC_F9,    KC_F10,    KC_F11,  KC_F12,   KC_PSCR,  KC_SCRL,  KC_PAUS,
        KC_GRV,   KC_1,     KC_2,     KC_3,     KC_4,     KC_5,     KC_6,     KC_7,     KC_8,     KC_9,     KC_0,     KC_MINS,   KC_EQL,  KC_BSPC,  XXXXXXX,  KC_HOME,  KC_PGUP,
        KC_TAB,   KC_Q,     KC_W,     KC_E,     KC_R,     KC_T,     KC_Y,     KC_U,     KC_I,     KC_O,     KC_P,     KC_LBRC,   KC_RBRC, KC_BSLS,  KC_DEL,   KC_END,   KC_PGDN,
        XXXXXXX,    KC_A,     KC_S,     KC_D,     KC_F,     KC_G,     KC_H,     KC_J,     KC_K,     KC_L,     KC_SCLN,  KC_QUOT,            KC_ENT,
        KC_LSFT,            KC_Z,     KC_X,     KC_C,     KC_V,     KC_B,     KC_N,     KC_M,     KC_COMM,  KC_DOT,   KC_SLSH,            KC_RSFT,            KC_UP,
        KC_LCTL,  KC_LWIN,  KC_LALT,                                KC_SPC,                                 KC_RALT,  FN_WIN,    KC_APP,  KC_RCTL,  KC_LEFT,  KC_DOWN,  KC_RGHT),

    [WIN_FN] = LAYOUT_tkl_ansi(
        QK_BOOT,            KC_BRID,  KC_BRIU,  KC_TASK,  KC_FILE,  UG_VALD,  UG_VALU,  KC_MPRV,  KC_MPLY,  KC_MNXT,  KC_MUTE,  KC_VOLD,  KC_VOLU,  _______,  _______,  UG_TOGG,
        _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,
        UG_TOGG,  UG_NEXT,  UG_VALU,  UG_HUEU,  UG_SATU,  UG_SPDU,  _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,
        OS_TOGGL, UG_PREV,  UG_VALD,  UG_HUED,  UG_SATD,  UG_SPDD,  _______,  _______,  _______,  _______,  _______,  _______,            _______,
        _______,            _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,  _______,            _______,            _______,
        _______,  GU_TOGG,  _______,                                _______,                                _______,  _______,  _______,  _______,  _______,  _______,  _______),
};
```

----

## keyboards/keychron/c3_pro_8k/ansi/keymaps/MY_MAPPING/rules.mk
- 新規作成：ファームウェアサイズの圧縮

```
VIA_ENABLE = yes
SPACE_CADET_ENABLE = no
GRAVE_ESC_ENABLE = no
LTO_ENABLE = yes
```

----

## あとはコンパイルと書き込み
コンパイルはQMK MSYSとかで→ `$ qmk compile -kb keychron/c3_pro_8k/ansi -km MY_MAPPING`

書き込みにはQMK Toolboxを……と言いたいところだが、**どういうわけかブートローダーに入ってもQMK Toolbox内で認識されない**（黄色文字が出ない）  
そこで解決策として（Windowsの場合）、

1. QMK Toolboxを管理者権限で開く
2. ウィンドウ上部の「Tools」タブを開いて「Install Drivers...」をクリック
3. `C:\Users\<自分のユーザー名>\AppData\Local\QMK\Toolbox`をコンソールで開く
4. （他にキーボードがなければ）Windowsのアクセシビリティ機能にある「スクリーンキーボード」をあらかじめ立ち上げておく
5. C3 Pro 8Kをブートローダーモードにする
6. `.\dfu-util.exe -a 0 --dfuse-address 0x08000000:leave -D <コンパイルしてできたファームウェアファイル（.bin）>`を実行
7. 書き込み完了
