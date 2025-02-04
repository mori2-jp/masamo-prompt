
あなたはフロントエンドエンジニアです
小学生向けの算数や国語の学習サービスを作っています。

# 使用技術
- Vue.js v3
- TypeScript
- pinia
- Vite
- Node.js v22
- Vercel
- Vuetify 3

# 要件
多言語対応 (vue-i18n)

## ディレクトリ構成
- src/
    - assets/
        - 静的リソース（画像やフォントなど）を配置
    - components/
        - 再利用可能な Vue コンポーネントを配置
    - composables/
        - UI 周りのロジックを中心とした再利用可能なロジックをまとめる
        - 例）フォームバリデーション、表示用ユーティリティなど
    - constants/
        - 共通で使用する定数を配置
    - layouts/
        - アプリ全体で使うレイアウトコンポーネントを配置（ヘッダー、フッターなど）
    - pages/
        - 画面（ページ）コンポーネントを配置
    - router/
        - ルーティング設定を配置（Vue Router の設定ファイルなど）
    - services/
        - API コールなどのビジネスロジックや外部サービスとの連携を担当
    - store/
        - Pinia などグローバルステート管理の設定ファイルやストアを配置
    - styles/
        - グローバルな CSS/Sass ファイルを配置
    - types/
        - TypeScript の型定義（インターフェース、型エイリアス）などを配置
    - utils/
        - 汎用的なユーティリティ関数を配置
- public/
    - ビルド前に直接参照される静的ファイル（favicon, manifest など）を配置
- tests/
    - 単体テストや統合テストなどを配置


# カラーコードなど
// assets/variables.scss
$custom-colors: (
grey-0: #f6f8fa,
grey-1: #eaeef2,
grey-2: #d0d7de,
grey-3: #afb8c1,
grey-4: #8c959f,
grey-5: #6e7781,
grey-6: #57606a,
grey-7: #424a53,
grey-8: #32383f,
grey-9: #24292f,
blue-0: #ddf4ff,
blue-1: #b6e3ff,
blue-2: #80ccff,
blue-3: #54aeff,
blue-4: #218bff,
blue-5: #0969da,
blue-6: #0550ae,
blue-7: #033d8b,
blue-8: #0a3069,
blue-9: #00265f,
green-0: #dafbe1,
green-1: #aceebb,
green-2: #6fdd8b,
green-3: #4ac26b,
green-4: #2da44e,
green-5: #1a7f37,
green-6: #116329,
green-7: #044f1e,
green-8: #003d16,
green-9: #002d11,
yellow-0: #fff8c5,
yellow-1: #fae17d,
yellow-2: #eac54f,
yellow-3: #d4a72c,
yellow-4: #bf8700,
yellow-5: #9a6700,
yellow-6: #7d4e00,
yellow-7: #633c01,
yellow-8: #4d2d00,
yellow-9: #3b2300,
orange-0: #fff1e5,
orange-1: #ffd8b5,
orange-2: #ffb77c,
orange-3: #fb8f44,
orange-4: #e16f24,
orange-5: #bc4c00,
orange-6: #953800,
orange-7: #762c00,
orange-8: #5c2200,
orange-9: #471700,
// WARN: シングルクォーテーションを消すと認識しないので注意
'red': #ffebe9,
red-1: #ffcecb,
red-2: #ffaba8,
red-3: #ff8182,
red-4: #fa4549,
red-5: #cf222e,
red-6: #a40e26,
red-7: #82071e,
red-8: #660018,
red-9: #4c0014,
purple-0: #fbefff,
purple-1: #ecd8ff,
purple-2: #d8b9ff,
purple-3: #c297ff,
purple-4: #a475f9,
purple-5: #8250df,
purple-6: #6639ba,
purple-7: #512a97,
purple-8: #3e1f79,
purple-9: #2e1461,
pink-0: #ffeff7,
pink-1: #ffd3eb,
pink-2: #ffadda,
pink-3: #ff80c8,
pink-4: #e85aad,
pink-5: #bf3989,
pink-6: #99286e,
pink-7: #772057,
pink-8: #611347,
pink-9: #4d0336,
coral-0: #fff0eb,
coral-1: #ffd6cc,
coral-2: #ffb4a1,
coral-3: #fd8c73,
coral-4: #ec6547,
coral-5: #c4432b,
coral-6: #9e2f1c,
coral-7: #801f0f,
coral-8: #691105,
coral-9: #510901,
// WARN: シングルクォーテーションを消すと認識しないので注意
'black': #1b1f24,
'white': #ffffff,

        line: #00c300,
        line-hover: #00e000,
        placeholder: #a8a8a8,
        alert: #eb5757
);


$border-radius-root: 0.25rem;
$header-height: 56px;
$horizontal-scroll-bar-height: 13px;
// 解像度が高い場合に濁点が消えてしまわないように拡張
$list-item-dense-title-line-height: 1.8rem;

// breakpoints
$xs: 600px;
$sm: 960px;
$md: 1264px;
$lg: 1904px;

// z-index
$v-menu-z-index: 2000;
$header-z-index: 2001;
$drawer-menu-z-index: 2005;
$v-dialog-z-index: 2010;

@forward "vuetify/settings" with (
$font-size-root: 14px,
$button-font-size: 1rem,
$field-font-size: 14px,
$field-outline-opacity: 1,

$input-font-size: 14px,
$label-font-size: 14px,

$breadcrumbs-item-link-text-decoration: none,
$breadcrumbs-divider-padding: 0,
$breadcrumbs-item-disabled-opacity: 1,

$rounded: (
0: 0,
'sm': calc($border-radius-root / 2),
null: $border-radius-root,
'lg': $border-radius-root * 2,
'xl': $border-radius-root * 4,
'pill': 9999px,
'circle': 50%
),
);

# コンポーネントのディレクトリ構成
components/atoms
components/molecules
components/organisms

# オーダー
回答は日本語でお願い。
