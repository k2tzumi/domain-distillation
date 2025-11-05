---
# You can also start simply with 'default'
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
# some information about your slides (markdown enabled)
title: アーキテクチャレベルで依存性を逆転させたら最高だった話
info: |
  PHPカンファレンス福岡2025
  https://phpcon.fukuoka.jp/2025/
# apply unocss classes to the current slide
class: text-center
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: fade-out
seoMeta:
  ogImage: auto
# duration of the presentation, default is '30min'
duration: 30min
# timer mode, can be 'countdown' or 'stopwatch', default is 'stopwatch'
timer: stopwatch
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
highlighter: shiki
css: unocss
colorSchema: dark
routerMode: hash
addons:
  - '@katzumi/slidev-addon-ogp-image'
  - slidev-addon-components
---

# アーキテクチャレベルで依存性を逆転させたら最高だった話

PHP カンファレンス福岡 2025 November 8, 2025.  
v0.0.2  
@katzumi(かつみ)

<div @click="$slidev.nav.next" class="mt-12 py-1" hover:bg="white op-10">
  Press Space for next page <carbon:arrow-right />
</div>

<div class="abs-br m-6 text-xl">
  <button @click="$slidev.nav.openInEditor" title="Open in Editor" class="slidev-icon-btn">
    <carbon:edit />
  </button>
  <a href="https://github.com/k2tzumi/domain-distillation" target="_blank" class="slidev-icon-btn">
    <carbon:logo-github />
  </a>
</div>

<div class="absolute bottom-4 right-4 opacity-75" style="z-index: -1;">
  <img src="https://phpcon.fukuoka.jp/2025/images/echo/4.png" class="h-60 w-auto" alt="公式ロゴ" />
</div>

<!--
本日は「アーキテクチャレベルで依存性を逆転させたら最高だった話」というタイトルで、お話しさせていただきます。
-->

---
transition: fade-out
layout: two-cols-header
class: text-left
---

# <carbon-user-avatar /> 自己紹介

katzumi（かつみ）と申します。

「障害のない社会をつくる」をビジョンとする「LITALICO（りたりこ）」に所属しています。
<a href="https://litalico.co.jp/" target="_blank">
  <img src="https://litalico.co.jp/ogp.png" class="w-40 mt-3 mb-8" />
</a>

以下のアカウントで活動しています。

::left::

<div class="flex items-center mb-6">
  <img src="https://pbs.twimg.com/profile_images/1961960367351005184/LpDD3fDz_400x400.jpg" class="rounded-full w-24 mr-6"/>

  <QRCode text="https://twitter.com/katzchum" class="w-28 h-28" />
</div>

<div class="flex items-center text-xl mt-4">
  <simple-icons-x class="mr-2" />
  <a href="https://twitter.com/katzchum" target="_blank" class="font-bold">katzchum</a>
</div>

::right::

<div class="flex flex-col space-y-4">
  <div class="flex items-center">
    <img src="https://avatars.githubusercontent.com/u/1182787?v=4" class="rounded-full w-24 mr-6"/>
    <div class="flex flex-col">
      <div class="flex items-center text-xl">
        <logos-github-octocat class="mr-2" /> 
        <a href="https://github.com/k2tzumi" target="_blank" class="font-bold">k2tzumi</a>
      </div>
      <div class="flex items-center text-xl mt-2">
        <simple-icons-zenn class="mr-2 text-green-400" />
        <a href="https://zenn.dev/katzumi" target="_blank" class="font-bold">katzumi</a>
      </div>
    </div>
  </div>
</div>

<style>
/* タイトルのスタイルはそのまま維持 */
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

<!--
みなさん、こんにちは。LITALICOの「かつみ」と申します。  
ご覧のXとGitHubで活動しています
-->

---
layout: two-cols-header
transition: fade-out
---

# <carbon-information /> お願い 🙏

写真撮影、SNS での実況について

登壇者の励みになるので是非ともご意見やご感想など、フィードバック頂けると助かります mm  
スライドの内容は、すでに以下の場所で公開されていますので、ぜひお手元でご覧ください  

* [forteeのプロポーザルページ](https://fortee.jp/phpcon-fukuoka-2025/proposal/6675eb22-13f1-4b11-8093-1b522345c2eb)
* または <fa6-brands-square-x-twitter /> の投稿

::left::

<Transform :scale="2.5">
　　　🙆‍♀📷<ph-projector-screen-chart-light /><br />
　　　🙅‍♂📹💸<br />
　　　🙅📸👨‍👦‍👦<br />
</Transform>

::right::

<br />
<Transform :scale="2">
<fa6-brands-square-x-twitter />
</Transform>
<br />
<a href="https://x.com/search?q=%23phpconfuk%20%23hall_d&f=live">#phpconfuk #hall_d</a>

<!--
まずはじめにお願いです。写真撮影、SNSでの実況、大歓迎です。スライドも公開済みですので、ぜひハッシュタグをつけて、ご意見やご感想をフィードバックいただけると励みになります。
-->

---

# <carbon-presentation-file /> 本日のお話すること
とある複雑なドメインを扱うシステムの物語です。集大成的なお話をします

* レセプト業務という複雑なドメインで、2 度の大規模法改正に立ち向かった実体験です。
* ２度のリアーキテクチャを経て Rezept as a Service を立ち上げた話（ドメイン蒸留）
* どうやってサービスを実現し、どういうメリットがあったのか？（戦術）
* 何故 Rezept as a Service が必要だと判断したのか？（戦略）

<v-click>

元ネタは昨年のアドベントカレンダーです。

<div max-w-130>

<a href="https://b.hatena.ne.jp/entrylist?url=https%3A%2F%2Fzenn.dev%2Flitalico%2Farticles%2Fdomain-distillation" target="_blank">
<img src="/hateb.png" />
</a>

</div>

</v-click>

<!--
本日は、とある複雑なドメインを扱うシステムの物語、その集大成的なお話をします。  
具体的には、「レセプト業務」という非常に複雑なドメインで、2度の大規模な法改正に立ち向かった実体験です。  
2回のリアーキテクチャを経て、「Rezept as a Service」という形でサービスを立ち上げた話をします。  
どうやってサービスを実現し、どんなメリットがあったのかという「戦術」面と、そもそも何故それが必要だと判断したのかという「戦略」面について、お話しします。
[click] 昨年のアドカレが元ネタになっています
-->

---
transition: fade-out
---

# <mdi-library /> レセプト業務の複雑さを視覚化

圧巻の 1.5K 頁オーバー。3 年に 1 回、大改訂（大改定）があります

<div max-w-170>

<img src="/ISBN-978-4805887332.png" />

</div>

<!--
まず初めに私たちが扱っている「レセプト業務」がどれだけ複雑か、視覚的にお見せします。これがその業務のルールブック、「事業者ハンドブック」です。  
どこのご家庭にもある「ちょうぜつ本」と比較しても、この圧倒的な厚さです。
-->

---
layout: image-right
image: /ISBN-978-4824300867.png
transition: slide-up
---

# <f7-square-stack-3d-up-fill /> 厚い本が更に厚くなる
手に取って持ち運べるような、携帯に便利な手引書感がまったくない、改訂版がこちら

なぜか３部作にｗ  
全体のページ数が 2k オーバーに  
モデルチェンジで x1.3 増ページはなかなか、、

<!--
しかも、改訂版でさらに厚くなり、なぜか3部作になりました。  
ページ数は2000ページを超えています。
-->

---

# <carbon-help /> 「レセプト業務」って何？
医療で言うところの診療報酬明細書（医療点数が書かれているアレ）を作成するお仕事です

<Transform :scale="1.2">

* **🧾 概要** 障害福祉サービスや介護事業所で提供されたサービスの費用を計算し、  
市町村や国保連などの支払い機関に請求する業務です。  
利用者は費用の一部（利用者負担額）を支払い、  
残りの公費・保険負担分を正確に計算し請求するのが重要な役割です。  
計算の基となるのは、サービス内容に応じて定められた「単位数」です。  
これは「単位数表」で定義されています。  

* **🗂️ 計算ルールをまとめたもの=単位数表を図式化したもの＝算定構造** 単位数を計算するプロセスを「算定」と呼びます。  
算定には国が定めた非常に細かくて複雑なルールがたくさんあります。  
これらの算定ルール及び、単位数表の関係を図式化したものを「算定構造」と呼びます。  
この算定構造は表形式でまとめられています。

</Transform>

<!--
「レセプト業務」とは何か。  
医療でいう診療報酬明細書、あの医療点数が書かれているアレに近いものを作成するお仕事です。  
障害福祉サービスで提供されたサービスの費用を計算し、市町村などに請求する業務です。  
利用者の負担額を計算し、残りの公費負担分を正確に計算・請求するのが重要な役割です。  
この計算ルールをまとめたものを「算定構造」と呼んでいます。  
先ほど見て頂いた分厚い本に、このルールとなる内容がまとめられています。
-->

---
layout: center
class: text-center
zoom: 2.0
---

# <carbon-bullhorn /> 宣伝

算定構造の作り方のエッセンスを紹介しました

<OgpImage url="https://www.docswell.com/s/katzumi/KLQL3Q-decision-table-implementation-tips" />

<!--
この算定構造の作り方をテーマにした発表をPHPカンファレンス関西でしました。もしよかったらご参照ください
-->

---

# <carbon-warning-alt /> 法改正の難しさ

大規模な法改正（報酬改定）が 3 年毎にあります

- 情報公開から実装完了まで約 3 ヶ月
- 算定ルール（報酬体系）が大幅に見直しされる
- 年々複雑化  
算定パターン数が 3 倍になるケースもある
- 過去 5 年間は再請求の為にロジックを残さないといけない
- 改正内容の影響範囲が広範囲にわたる  
改正は請求（報酬単価の見直し）だけでなく、支援内容・実績記録すべてを変える  
→ システム全体が連鎖的に影響を受ける

<!--
この業務には、3年ごとに大規模な法改正があります。  
これが非常に厄介で、情報公開から実装完了まで約3ヶ月しかありません。  
算定ルールは大幅に見直され、年々複雑化します。  
しかも、過去5年分は再請求のために古いロジックを残さなければなりません。
影響範囲は請求だけでなく、実績記録などシステム全体に及びます。
-->

---

# <carbon-flow /> レセプト業務のプロセス特有の構造上の問題
データの流れ = 依存の方向となる

<img src="/rezept-data-flow.svg" />

<!--
さらに、レセプト業務特有の構造上の問題があります。  
「レセプト業務」は、その計算根拠となる上流の「各種実績データ（体制状況、サービス利用等）」に完全に依存する構造となっています。データフローの方向がそのままシステムの依存関係に直結しており、上流データが確定しなければ下流の請求処理（レセプト業務）が進められない「直列的なデータ依存構造」がありました。
-->

---
layout: two-cols-header
---

# <carbon-face-dissatisfied /> レセプトの開発の辛み
この矛盾にお気づき頂けますでしょうか。。

レセプト業務は:
- ビジネスの生命線
- 最も複雑でミスの影響が大きい
- <span v-mark="{ at: 3, color: 'orange', type: 'highlight' }">パターン数が多く、テストに時間がかかる</span>
- 最も安定させるべき

<v-click at="1">

なのに、「下流」に位置している。

</v-click>

::left::

<v-click at="1">


上流（実績データ）:
- 各プロダクトが使いやすさを追求
- <span v-mark="{ at: 2, color: 'orange', type: 'circle' }">仕様確定に時間がかかる</span>
- 変更が頻繁

</v-click>

::right::

<v-click at="1">

下流（レセプト）:
- <span v-mark="{ at: 4, color: 'orange', type: 'underline' }">上流の影響を直接受ける</span>  
算定ルールが実績データと密結合となる
- 最も不安定な位置に

</v-click>

<!--
ここに大きな「矛盾」があります。  
レセプト業務は、事業所の収益に直結する不可欠な業務です。複雑性が高く、ミスが経営リスクにつながるため、最も安定させる必要があります。   
[click] 「なのに」、データの流れの「下流」に位置してしまっています。  
[click] 一方、上流の実績データは、使いやすさを追求するため、「仕様確定に時間がかかり」、「変更が頻繁」に発生します。  
[click] パターンも多く、検証に時間もかかります。  
[click] 結果として、レセプト業務が、上流のデータ変更の影響を直接受けてしまい、不安定化しやすい構造になっていました。
-->

---

# <carbon-warning-alt /> 法改正の難しさ（再掲）

大規模な法改正（報酬改定）が 3 年毎にあります

- <span v-mark="{ at: 2, color: 'orange', type: 'highlight' }">情報公開から実装完了まで約 3 ヶ月</span>
- 算定ルール（報酬体系）が大幅に見直しされる
- 年々複雑化  
算定パターン数が 3 倍になるケースもある
- 過去 5 年間は再請求の為にロジックを残さないといけない
- <span v-mark="{ at: 1, color: 'orange', type: 'highlight' }">改正内容の影響範囲が広範囲にわたる</span>  
改正は請求（報酬単価の見直し）だけでなく、支援内容・実績記録すべてを変える  
→ システム全体が連鎖的に影響を受ける

<v-clicks>

<div class="speech-bubble">
改修ボリューム的に1チームで完結しない。関心ごとも様々で混在し絡み合っている
</div>

</v-clicks>
<v-clicks at=2>
<div class="speech-bubble">
実績データの仕様Fixをまってからレセプト機能を実装していたら、間に合わなくなる！
</div>
</v-clicks>

<!--
レセプト開発の難しさに加えて、更に法改正の難しさが掛け合わされます。      
[click] 改正内容の影響範囲が広範囲にわたり、1チームで完結しないボリュームとなります。  
依存している機能を開発しているチームと連携を行う必要があります。    
[click] ただ実績データの仕様の確定を待っていられない、時間的な制約もあります。
-->

---
layout: center
class: text-center
zoom: 2.0
---

# 😰 どうすれば。。

<!--
法改正は、施行日が国によって固定されており、加えて毎回内容が異なるため、対応に多大な労力を強いられます。
-->

---
layout: center
class: text-center
zoom: 1.5
---

# <carbon-time /> 2020年から4年半の取り組みを紹介

<!--
ここから、私たちが4年半かけて行ってきた取り組みを紹介します。
-->

---
layout: center
class: text-center
zoom: 2.0
---

# <carbon-summary-kpi /> 先にまとめ
2 つの Key Success Factor

<v-clicks>

1. 関心ごとの分離
2. 依存性逆転

</v-clicks>

<!--
先に結論を言うと、Key Success Factor、つまり成功の鍵は
[click]「関心ごとの分離」と
[click] 「依存性逆転」の2つでした。
-->

---
zoom: 1.5
---

# <carbon-cut-out /> 関心ごとの分離
責務の分離、Separation Of Concerns／SOC

* 責務を十分小さくする  
修正した場合に影響範囲を限定的にする
* 認知できるサイズとなるように分離する  
* 修正する理由は 1 つにする

<!--
まず、2021年度の法改正で取り組んだのが「関心ごとの分離」です。  
SOLID原則の一つ、責務分離です。  
「機能や関心ごとをしっかり分け、一つの部品に一つの役割だけを持たせましょう」というやつです。
-->

---
layout: two-cols-header
---

# <carbon-cut-out /> どういう関心ごとを分離していったのか？
まずは 2 つの関心ごとを分離した

* 法改正で旧制度と新制度のロジックが混ざらないようにした  
同一の関心ごとを時系列で分断させ別の関心ごととして扱う
* 法改正の影響がない周辺の関心ごとを外に出した  
時系列で変化をしない処理（請求状態や帳票管理等）を独立させた

<v-click at="1">

<div class="speech-bubble">
2021年度法改正での取り組み
</div>

</v-click>

::left::

<v-click at="1">

<OgpImage url="https://zenn.dev/litalico/articles/confronting-law-amends-with-microservices" />

</v-click>

::right::

<v-click at="1">

<OgpImage url="https://zenn.dev/litalico/articles/confronting-law-amends-with-microservices" />

</v-click>

<!--
具体的には2つのことを行いました。
1つは、法改正で旧制度と新制度のロジックが混ざらないよう、時系列で分断しました。  
2つ目は、法改正の影響がない請求状態の管理など、周辺の関心ごとを外に切り出しました。  　　
[click] 詳細な取り組みの内容を記事にまとめていますので、そちらをご参照ください。
-->

---
class: py-10
glowSeed: 215
---

# <carbon-chart-3d /> 関心ごとの分離によるメリット

<span>法改正への対応とシステムの健全性向上</span>

<div mt-8 />

<div flex flex-col items-center>
  <div
    grid grid-cols-3 gap-6
    w-full max-w-200
  >
    <div
      v-click="1"
      border="2 solid yellow-600" bg="yellow-600/20"
      rounded-lg overflow-hidden
      transition duration-500 ease-in-out
      :class="$clicks < 1? 'opacity-0 translate-y-20' : 'opacity-100 translate-y-0'"
    >
      <div bg="yellow-600/40" px-4 py-2 flex items-center justify-center text-sm>
        <span font-bold>実装効率の向上 🚀</span>
      </div>
      <div px-4 py-4 flex flex-col gap-3>
        <div flex items-center gap-3>
          <div i-carbon:add-alt text-3xl w-10 text-yellow-400 />
          <div>
            <div font-semibold>新規ロジックのみを追加</div>
            <div text-xs text-zinc-400>旧制度のコードに触れない。if文を追加しない</div>
          </div>
        </div>
        <div flex items-center gap-3>
          <div i-carbon:center-circle text-3xl w-10 text-yellow-400 />
          <div>
            <div font-semibold>新制度に集中して実装</div>
            <div text-xs text-zinc-400>旧制度の知識が不要</div>
          </div>
        </div>
      </div>
    </div>
    <div
      v-click="2"
      border="2 solid green-600" bg="green-600/20"
      rounded-lg overflow-hidden
      transition duration-500 ease-in-out
      :class="$clicks < 2 ? 'opacity-0 translate-y-20' : 'opacity-100 translate-y-0'"
    >
      <div bg="green-600/40" px-4 py-2 flex items-center justify-center text-sm>
        <span font-bold>テスト範囲の限定 🛡️</span>
      </div>
      <div px-4 py-4 flex flex-col gap-3>
        <div flex items-center gap-3>
          <div i-carbon:checkmark-outline w-10 text-green-400 />
          <div>
            <div font-semibold text-base>新規ロジックのみをテスト</div>
            <div text-xs text-zinc-400>旧制度ロジックへの影響がない</div>
          </div>
        </div>
        <div flex items-center gap-3>
          <div i-carbon:locked text-3xl w-10 text-green-400 />
          <div>
            <div font-semibold text-base>不変性の確保</div>
            <div text-xs text-zinc-400>ロジックの安定性が向上</div>
          </div>
        </div>
      </div>
    </div>
    <div
      v-click="3"
      border="2 solid blue-600" bg="blue-600/20"
      rounded-lg overflow-hidden
      transition duration-500 ease-in-out
      :class="$clicks < 3 ? 'opacity-0 translate-y-20' : 'opacity-100 translate-y-0'"
    >
      <div bg="blue-600/40" px-4 py-2 flex items-center justify-center text-sm>
        <span font-bold>システムの保守性向上 🧹</span>
      </div>
      <div px-4 py-4 flex flex-col gap-3>
        <div flex items-center gap-3>
          <div i-carbon:trash-can text-3xl text="white" w-10 text-blue-400 />
          <div>
            <div font-semibold text-base>不要ロジックの安全な削除</div>
            <div text-xs text-zinc-400>役割を終えたコードを除去</div>
          </div>
        </div>
        <div flex items-center gap-3>
          <div i-carbon:growth text-3xl w-10 text-blue-400 />
          <div>
            <div font-semibold text-base>複雑さの増大を防止</div>
            <div text-xs text-zinc-400>システムの健全性を維持</div>
          </div>
        </div>
      </div>
    </div>
  </div>

  <div
    v-click="4"
    mt-6 flex justify-center
    transition duration-500 ease-in-out
    :class="$clicks < 4 ? 'opacity-0 scale-90' : 'opacity-100 scale-100'"
  >
    <div
      bg="gray-700/80"
      rounded-lg py-3 px-6 text-center flex items-center gap-3 text-white
    >
      <div i-carbon:connection text-white text-2xl />
      <div>
        <div text-xl font-bold>モジュールとして独立させやすくなった ✂️</div>
        <div text-sm mt-1>責務の明確化により、将来的にモジュールやサービスとして独立させやすくなった</div>
      </div>
    </div>
  </div>
</div>

<!--
この「関心ごとの分離」によるメリットは3つあります。  
[click] まず、実装効率の向上です。旧制度のコードに触れず、新制度のロジック追加に集中できます。  
[click] 次に、テスト範囲が限定的になり、新規ロジックのみをテストすればよくなります。  
[click] そして、保守性の向上です。役割を終えた旧制度のロジックを安全に削除でき、システムの複雑化を防げます。  
[click] モジュールとして独立させやすくなりました  
責務が明確になって、サービスとして切り出しやすくなりました。
-->

---
layout: center
class: text-center
---

# <carbon-task-complete /> リアーキテクチャは成功 ✨

関心ごとの分離により、1 つのサービスの安定性は大きく改善されました。

<div mt-10 />

<div grid grid-cols-3 gap-8 w-2xl m-auto>

  <div border="2 solid green-600" bg="green-600/20" rounded-lg p-4>
    <div i-carbon:edit text-4xl text-green-400 mb-2 />
    <span font-bold text-lg>変更容易性 / 理解容易性</span>
    <hr class="my-2 border-green-500">
    <p text-sm>旧制度に触れず新ロジックを追加可能になり、認知負荷が軽減。</p>
  </div>

  <div border="2 solid blue-600" bg="blue-600/20" rounded-lg p-4>
    <div i-carbon:test-tool text-4xl text-blue-400 mb-2 />
    <span font-bold text-lg>テスト容易性</span>
    <hr class="my-2 border-blue-500">
    <p text-sm>修正ロジックのみをテストすれば良くなり、ロジックの不変性が確保された。</p>
  </div>

  <div border="2 solid yellow-600" bg="yellow-600/20" rounded-lg p-4>
    <div i-carbon:clean text-4xl text-yellow-400 mb-2 />
    <span font-bold text-lg>保守性の向上</span>
    <hr class="my-2 border-yellow-500">
    <p text-sm>不要なロジックを安全に削除可能になり、複雑さの増大を防止。</p>
  </div>

</div>

<div v-click="1"
  class="absolute top-0 left-0 w-full h-full pointer-events-none" 
  style="z-index: 100;"
>
  <div v-motion
    :initial="{ x: -200, y: 800, scale: 0.5, rotate: 90, opacity: 0 }"
    :enter="{ x: 0, y: -800, scale: 1.5, rotate: 0, opacity: 1 }"
    :delay="50"
    :duration="3000"
    class="absolute text-5xl" 
    style="left: 10%; bottom: 0;">👏</div>
  
  <div v-motion
    :initial="{ x: 0, y: 800, scale: 1, rotate: 0, opacity: 0 }"
    :enter="{ x: 0, y: -900, scale: 2, rotate: 360, opacity: 0.8 }"
    :delay="300"
    :duration="3500"
    class="absolute text-6xl" 
    style="left: 50%; bottom: 0;">🎉</div>
  
  <div v-motion
    :initial="{ x: 200, y: 800, scale: 0.5, rotate: -90, opacity: 0 }"
    :enter="{ x: 0, y: -700, scale: 1.5, rotate: 0, opacity: 1 }"
    :delay="600"
    :duration="2800"
    class="absolute text-5xl" 
    style="left: 80%; bottom: 0;">👏</div>

  <div v-motion
    :initial="{ x: -50, y: 800, scale: 0.8, rotate: 45, opacity: 0 }"
    :enter="{ x: 50, y: -850, scale: 1.2, rotate: 200, opacity: 0.7 }"
    :delay="100"
    :duration="3200"
    class="absolute text-4xl" 
    style="left: 40%; bottom: 0;">✨</div>
    
  <div v-motion
    :initial="{ x: 300, y: 800, scale: 1, rotate: -30, opacity: 0 }"
    :enter="{ x: -100, y: -950, scale: 1.8, rotate: 10, opacity: 0.9 }"
    :delay="500"
    :duration="3600"
    class="absolute text-6xl" 
    style="left: 65%; bottom: 0;">🥳</div>

</div>

<!--
この第1弾のリアーキテクチャは成功し、サービスの変更容易性やテスト容易性が大きく改善されました。  
[click] はい！ぱちぱちぱちー
-->

---
class: flex justify-center items-center gap-20 px40 text-xl
---

<div text-6xl absolute :class="$clicks < 1 ? 'text-white' : 'translate-y-[-2rem] scale-40 text-white/30'" transition duration-500 ease-in-out>
  <span>🫵 勝ったッ！ 第１部完！ 🎉</span>
</div>

<div flex flex-col items-center>

<v-clicks>

<div mt-20>

<h1 flex items-center text="4xl!">
  <p>...と、この時は思っていました。</p>
</h1>

</div>

</v-clicks>

</div>

<!--
勝ったッ！第1部完！  
[click] …と、この時は思っていました。
-->

---
layout: center
class: text-center
---

# <carbon-ibm-data-product-exchange /> 外部環境の変化と新たな課題

法改正を乗り越えて安心していたら今度は**ビジネスが拡大**しました。

<div mt-10 />

<div flex items-center justify-center gap-10>
  
  <div
    class="w-1/3 transition-opacity duration-500 ease-in-out"
  >
    <div i-carbon:building text-6xl text-blue-400 m-auto mb-4 />
    <h3 font-bold text-xl>ビジネスが拡大</h3>
    <p text-sm mt-2>プロダクトが増加し、カバーすべきサービスの領域が一気に増えた（重複もあった）</p>
  </div>

  <div 
    v-click="1"
    class="transition-opacity duration-500 ease-in-out"
    :class="$clicks < 1 ? 'opacity-0' : 'opacity-100'"
    i-carbon:arrow-right text-5xl text-gray-500
  />

  <div 
    v-click="1"
    class="w-1/3 transition-opacity duration-500 ease-in-out"
    :class="$clicks < 1 ? 'opacity-0 scale-90' : 'opacity-100 scale-100'"
  >
    <div i-carbon:group text-6xl text-red-400 m-auto mb-4 />
    <h3 font-bold text-xl>対応領域の拡大と関係者の分散</h3>
    <p text-sm mt-2>法改正でカバーすべき対応領域が広がる<br />関心ごとや担当者がシステム間で分散・重複し始めた。</p>
  </div>
</div>

<div mt-10 />

<span 
  v-click="1"
  class="transition-opacity duration-500 ease-in-out"
  :class="$clicks < 1 ? 'opacity-0' : 'opacity-100'"
  text-xl font-bold text-red-300>システム全体の複雑さ（協調コスト）が急上昇
</span>

<!--
法改正対応の後にビジネスが拡大して、新たな課題がでてきました。    
[click] プロダクト増加により、対応領域が急拡大し、担当者や関心ごとがシステム間で分散・重複しました。結果、システム全体の整合性を取るための協調コストが急増して、法改正の難易度が更に上昇しました。
-->

---
layout: center
class: text-center
---

# ⚠️ まだまだ足りない！ (限界突破の必要性)

一サービスの改善がされましたが、**全社視点**では個別対応での限界が見えました

<div mt-10 />

<div flex justify-center items-center>
  <div
    border="4 solid red-500" bg="red-500/10"
    rounded-xl p-10 shadow-2xl
  >
    <div i-carbon:roadblock text-8xl text-red-400 m-auto />
    <h2 font-extrabold text-4xl text-red-300 mt-4>「個別最適」のままでは限界！</h2>
    <p text-xl mt-4>
      ✅　特定サービスのみの <span style="color: lime; font-weight: bold;">変更容易性</span> は確保した<br />
      🛑　<span style="color: yellow; font-weight: bold;">複数サービス</span>での安定した法改正対応を実現したい<br />
      🛑 関係者が多く、<span style="color: orange; font-weight: bold;">全社的なルール統一</span> の <span style="color: red; font-weight: bold;">難易度が極めて高い</span><br />
      🛑 レセプト業務に精通した<span style="color: gray; font-weight: bold;">エンジニアが圧倒的に不足！</span>
    </p>
    <p text-2xl font-bold mt-6>
      <span text-yellow-300>法改正を乗り切るための、</span><br />
      <span text-red-300>次のレベルの構造化が必要!</span>
    </p>
  </div>
</div>

<!--
現在のシステムはサービスごとの個別最適で、法改正対応を複数サービスで行うには再現性を担保できないと考えました。  
全社的なルール統一の難しさや専門エンジニア不足が限界のため、次レベルの構造化が必要と判断しました。
-->

---
layout: center
class: text-center
zoom: 1.8
---

# <arcticons-emoji-electric-light-bulb /> 依存性逆転という解決
タイトル回収

依存性逆転の原則（Dependency Inversion Principle）を  
アーキテクチャレベルで適用する

<!--
ここで、本日のタイトルである「依存性逆転の原則」をアーキテクチャレベルで適用します。
-->

---
layout: center
class: text-center
zoom: 1.9
---

# <carbon-receipt /> Rezept as a Service
レセプト業務を BaaS (Backend as a Service) 化

<!--
具体的には、レセプト業務をBaaS(Backend as a Service)化した、「Rezept as a Service」です。
-->

---
layout: center
class: text-center
---

# <carbon-arrows-horizontal /> 発想の転換 - 依存性逆転

<div mt-8 text-2xl>
  重要な気づき：
  <span v-click="1" class="font-bold text-yellow-300">データの流れは変えられない。</span><br />
  <span v-click="2" class="font-bold text-green-300">でも、依存の方向は変えられる！</span>
</div>

<div class="w-full h-80 grid grid-cols-2 gap-8 mt-10">

  <div
    v-click="3"
    class="flex flex-col items-center justify-center p-6 bg-gray-800/70 rounded-lg shadow-xl border-l-4 border-yellow-500 transition duration-500 ease-in-out"
    :class="$clicks < 3 ? 'opacity-0 scale-90' : 'opacity-100 scale-100'"
  >
    <h3 class="text-3xl font-bold text-yellow-300 mb-4">データの流れ (事実)</h3>
    <p class="text-xl">実績データは「支援アプリ」から「レセプトアプリ」へ流れる</p>
    <p class="text-lg italic mt-2 text-yellow-200">→ これは変えられない</p>
  </div>

  <div
    v-click="4"
    class="flex flex-col items-center justify-center p-6 bg-gray-800/70 rounded-lg shadow-xl border-r-4 border-green-500 transition duration-500 ease-in-out"
    :class="$clicks < 4 ? 'opacity-0 scale-90' : 'opacity-100 scale-100'"
  >
    <h3 class="text-3xl font-bold text-green-300 mb-4">依存の方向 (設計)</h3>
    <p class="text-xl">設計上、「どちらがどちらに依存するか」</p>
    <p class="text-lg italic mt-2 text-green-300">→ これは設計で変えられる!</p>
  </div>
</div>

<div 
  v-click="5"
  class="absolute inset-0 z-20 flex items-center justify-center bg-black/50 backdrop-blur-sm transition duration-700 ease-in-out"
  :class="$clicks < 5 ? 'opacity-0 pointer-events-none' : 'opacity-100 pointer-events-auto'"
>
  <div 
    class="p-10 bg-white/10 backdrop-blur-lg border-4 border-yellow-400 rounded-2xl shadow-2xl transform scale-125 transition duration-500"
  >
    <h3 class="text-6xl font-extrabold text-white mb-4">依存性逆転の核心</h3>
    <p class="text-4xl text-yellow-300 font-bold tracking-wide">
      データの流れ <span text-red font-bold>≠</span> 依存の方向
    </p>
  </div>
</div>

<!--
ここで重要な「発想の転換」をしました。  
[click]「データの流れ」は変えられません。  
[click]「でも、依存の向きは変えられる！」  
[click] 実績データがレセプト業務の前段で作られるという「データの流れ」と、[click] 設計上「どちらがどちらに依存するか（設計）」は、[click] イコールではない「変えることが出来る」という考え方です。
-->

---
layout: default
---

# <carbon-diagram /> 実績データ → レセプト業務の依存関係

<Transform :scale="0.7">

<img src="/rezept-as-a-service.svg" />


</Transform>

<!--
新しい構造「Rezept as a Service」では、レセプト業務の算定ロジック（上位モジュール）は、下位のプロダクト（A/B/C）が持つ実績データには直接依存しません。

代わりに、算定ロジックは、データの構造を定めた契約である「APIインターフェース」にのみ依存します。  
各プロダクトはインターフェースの契約に従って実装を行うことで、算定ロジックは実績データの詳細実装から隔離され、より安定し、変更に強い構造になります。  
これが依存性逆転です。
-->

---
layout: center
class: text-center
---

# <carbon-analytics /> 依存性逆転の効果 ✨

<div mt-8 text-2xl>
  発想の転換によって、何が変わったか？
</div>

<div class="w-full h-90 grid grid-cols-2 gap-8 mt-10">

  <div
    class="flex flex-col p-6 bg-gray-800/70 rounded-lg shadow-xl border-l-4 border-red-500 transition duration-500 ease-in-out"
  >
    <h3 class="text-3xl font-bold text-red-300 mb-4">🔴 Before (不安定な状態)</h3>
    <ul class="list-disc list-inside text-xl space-y-3 text-left">
      <li>レセプト業務はデータの<span class="font-bold">下流</span>に位置</li>
      <li>実績データと<span class="font-bold">密結合</span>（<span class="font-bold">不安定</span>）</li>
      <li>上流の仕様変更に<span class="font-bold">常に振り回される</span></li>
      <li>実績仕様が<span class="font-bold">確定待ち</span></li>
      <li>開発が<span class="font-bold">直列</span>になり、スピードが出ない</li>
    </ul>
  </div>

  <div
    v-click="1"
    class="flex flex-col p-6 bg-gray-800/70 rounded-lg shadow-xl border-r-4 border-green-500 transition duration-500 ease-in-out"
    :class="$clicks < 1 ? 'opacity-0 scale-90' : 'opacity-100 scale-100'"
  >
    <h3 class="text-3xl font-bold text-green-300 mb-4">🟢 After (安定・自立した状態)</h3>
    <ul class="list-disc list-inside text-xl space-y-3 text-left">
      <li>レセプト業務を<span class="font-bold">コア業務としてサービス化</span></li>
      <li>他に依存しない状態（<span class="font-bold">安定</span>）</li>
      <li>実績データの詳細から<span class="font-bold">完全に切り離される</span></li>
      <li>仕様を<span class="font-bold">先行確定</span>し、開発開始可能に</li>
      <li>開発が<span class="font-bold">並行</span>になり、リリースが早まる</li>
    </ul>
  </div>
</div>

<div 
  v-click="2"
  class="absolute inset-0 z-20 flex items-center justify-center bg-black/50 backdrop-blur-sm transition duration-700 ease-in-out"
  :class="$clicks < 2 ? 'opacity-0 pointer-events-none' : 'opacity-100 pointer-events-auto'"
>
  <div 
    class="p-10 bg-white/10 backdrop-blur-lg border-4 border-green-400 rounded-2xl shadow-2xl transform scale-125 transition duration-500"
  >
    <h3 class="text-5xl font-extrabold text-white mb-4">最も重要な成果</h3>
    <p class="text-2xl text-green-300 font-bold tracking-wide leading-relaxed">
      コアな部分を、<br />
      <span class="text-yellow-300">最も安定した位置</span>に置くことができた
    </p>
  </div>
</div>

<!--
この依存性逆転で何が変わったか。  
Beforeの状態は、レセプト業務がデータの下流に位置し、上流の仕様変更に振り回され、開発が直列的でした。  
[click] Afterの状態は、レセプト業務がコア業務としてサービス化され、他に依存しない「安定」した状態になりました。
実績データの詳細実装から切り離され、仕様を「先行確定」し、開発を「並行」して進められるようになりました。  
[click] 最も重要な成果は、ビジネス上のコアとなる部分を、システムの中で「最も安定した位置に置くことができた」ということです。
-->

---
layout: center
class: text-center
zoom: 2.0
---

# <carbon-face-activated /> 何が最高だったのか？
当初想定していた以上の効果

<!--
では、具体的に何が「最高」だったのか？  
当初の想定以上の効果が4つありました。
-->

---
layout: center
class: text-center
---

# <carbon-text-clear-format /> 1. 責務が明確になり、関心ごとにのみ集中 🎯

<div mt-8 text-2xl>
  この効果は、実績データを蓄積するクライアント（支援アプリなど）にも波及
</div>

<div class="w-full h-90 grid grid-cols-2 gap-8 mt-10">

  <div
    class="flex flex-col p-6 bg-gray-800/70 rounded-lg shadow-xl border-l-4 border-red-500 transition duration-500 ease-in-out"
  >
    <h3 class="text-3xl font-bold text-red-300 mb-4">🔴 Before (外部依存)</h3>
    <ul class="list-disc list-inside text-xl space-y-3 text-left">
      <li>常に<span class="font-bold">実績データの仕様変更</span>など、<span class="font-bold">外部の動向</span>に注意</li>
      <li>レセプト業務（算定基準）以外の<span class="font-bold">色々なこと</span>に気を配る必要</li>
      <li>責務が曖昧になり、<span class="font-bold">開発の集中力を欠く</span></li>
    </ul>
  </div>

  <div
    v-click="1"
    class="flex flex-col p-6 bg-gray-800/70 rounded-lg shadow-xl border-r-4 border-green-500 transition duration-500 ease-in-out"
    :class="$clicks < 1 ? 'opacity-0 scale-90' : 'opacity-100 scale-100'"
  >
    <h3 class="text-3xl font-bold text-green-300 mb-4">🟢 After (本質集中)</h3>
    <ul class="list-disc list-inside text-xl space-y-3 text-left">
      <li><span class="font-bold">レセプト業務の本質的な責務</span>（算定基準）にのみ集中</li>
      <li>外部の動向を<span class="font-bold">一切気にしなくていい</span>状態に</li>
      <li>責務が明確になり、<span class="font-bold">プロダクト開発に全力投球</span></li>
    </ul>
  </div>
</div>

<div 
  v-click="2"
  class="absolute inset-0 z-20 flex items-center justify-center bg-black/50 backdrop-blur-sm transition duration-700 ease-in-out"
  :class="$clicks < 2 ? 'opacity-0 pointer-events-none' : 'opacity-100 pointer-events-auto'"
>
  <div 
    class="p-10 bg-white/10 backdrop-blur-lg border-4 border-yellow-400 rounded-2xl shadow-2xl transform scale-125 transition duration-500"
  >
    <h3 class="text-5xl font-extrabold text-white mb-4">成果の鍵</h3>
    <p class="text-2xl text-yellow-300 font-bold tracking-wide leading-relaxed">
      <span class="text-green-300">スキーマの定義</span>ができれば、<br />
      <span class="font-bold">双方、開発に全力集中できる</span>
    </p>
  </div>
</div>

<!--
1つ目。  
以前は、関心ごとも混在し絡み合っており、幅広く影響を見ないといけませんでした。  
それぞれお互いの改修内容を気にする必要がありました。  
[click] それが「責務が明確になり、自身の関心ごとにのみ集中できる」様になりました。  
[click] 高品質なAPIスキーマさえ定義できれば、レセプトチームとプロダクトチームは、お互いの状況を気にせずに開発に集中することができます。
-->

---
layout: center
class: text-center
---

# <carbon-test-tool /> 2. I/F仕様書に向き合ってテストできる 🧪

<div mt-8 text-2xl>
  依存性逆転は、<span class="font-bold">テスト設計</span>にも大きな効果がある
</div>

<div class="w-full h-90 grid grid-cols-2 gap-8 mt-10">

  <div
    class="flex flex-col p-6 bg-gray-800/70 rounded-lg shadow-xl border-l-4 border-red-500 transition duration-500 ease-in-out"
  >
    <h3 class="text-3xl font-bold text-red-300 mb-4">🔴 Before (不安定なテスト)</h3>
    <ul class="list-disc list-inside text-xl space-y-3 text-left">
      <li><span class="font-bold">実績データの詳細なパターン</span>を常に考慮する必要</li>
      <li>テストの<span class="font-bold">境界が曖昧</span>になりがち</li>
      <li>結合テストが主体となり、<span class="font-bold">検証コストが高い</span></li>
    </ul>
  </div>

  <div
    v-click="1"
    class="flex flex-col p-6 bg-gray-800/70 rounded-lg shadow-xl border-r-4 border-green-500 transition duration-500 ease-in-out"
    :class="$clicks < 1 ? 'opacity-0 scale-90' : 'opacity-100 scale-100'"
  >
    <h3 class="text-3xl font-bold text-green-300 mb-4">🟢 After (安定したテスト)</h3>
    <ul class="list-disc list-inside text-xl space-y-3 text-left">
      <li><span class="font-bold">インターフェース仕様</span>に基づいた厳密なテストケース設計が可能</li>
      <li><span class="font-bold">テストの境界が明確</span>になり、検証が容易に</li>
      <li>全体の<span class="font-bold">品質担保がより確実</span>に</li>
      <li>外部に依存しない<span class="font-bold">ユニットテストの増加</span></li>
    </ul>
  </div>
</div>

<div 
  v-click="2"
  class="absolute inset-0 z-20 flex items-center justify-center bg-black/50 backdrop-blur-sm transition duration-700 ease-in-out"
  :class="$clicks < 2 ? 'opacity-0 pointer-events-none' : 'opacity-100 pointer-events-auto'"
>
  <div 
    class="p-10 bg-white/10 backdrop-blur-lg border-4 border-green-400 rounded-2xl shadow-2xl transform scale-125 transition duration-500"
  >
    <h3 class="text-5xl font-extrabold text-white mb-4">テストの自立</h3>
    <p class="text-2xl text-green-300 font-bold tracking-wide leading-relaxed">
      外部の事情に左右されず、<br />
      <span class="text-yellow-300 font-bold">独立してテストを行うことができる</span>
    </p>
  </div>
</div>

<!--
2つ目。   
以前は、実績データの蓄積時にどの様なバリデーションがあるか？など様々なパターンを意識してテストケースを考えていました。  
[click] それが、「I/F仕様書に向き合ってテスト設計を考える」ということが可能になりました。    
[click] 外部の事情に左右されず、契約、つまりインターフェースに基づいた厳密なテストが可能になりました。  
「コア業務単体のみで、独立してテストができる」ようになりました。
-->

---
layout: center
class: text-center
---

# <carbon-chat-bot /> 3. コミュニケーションコストを最低限に⬇️

<div mt-8 text-2xl>
  依存性逆転は、<span class="font-bold">組織的な開発効率</span>も大きく改善
</div>

<div class="w-full h-90 grid grid-cols-2 gap-8 mt-10">

  <div
    class="flex flex-col p-6 bg-gray-800/70 rounded-lg shadow-xl border-l-4 border-red-500 transition duration-500 ease-in-out"
  >
    <h3 class="text-3xl font-bold text-red-300 mb-4">🔴 Before (高い調整コスト)</h3>
    <ul class="list-disc list-inside text-xl space-y-3 text-left">
      <li><span class="font-bold">常に他チームとの調整</span>が必要</li>
      <li><span class="font-bold">細かい仕様の確認や質問</span>が頻繁に発生</li>
      <li>依存関係による<span class="font-bold">待ち時間</span>が発生</li>
    </ul>
  </div>

  <div
    v-click="1"
    class="flex flex-col p-6 bg-gray-800/70 rounded-lg shadow-xl border-r-4 border-green-500 transition duration-500 ease-in-out"
    :class="$clicks < 1 ? 'opacity-0 scale-90' : 'opacity-100 scale-100'"
  >
    <h3 class="text-3xl font-bold text-green-300 mb-4">🟢 After (自立と集中)</h3>
    <ul class="list-disc list-inside text-xl space-y-3 text-left">
      <li><span class="font-bold">Rezept as a Service</span>として提供</li>
      <li>各プロダクトチームは<span class="font-bold">自身のペースで開発可能</span></li>
      <li><span class="font-bold">ドキュメント</span>を見て自分で実装・デプロイ</li>
      <li><span class="font-bold">不要な調整が減り</span>、本質的な開発に集中</li>
    </ul>
  </div>
</div>

<div 
  v-click="2"
  class="absolute inset-0 z-20 flex items-center justify-center bg-black/50 backdrop-blur-sm transition duration-700 ease-in-out"
  :class="$clicks < 2 ? 'opacity-0 pointer-events-none' : 'opacity-100 pointer-events-auto'"
>
  <div 
    class="p-10 bg-white/10 backdrop-blur-lg border-4 border-green-400 rounded-2xl shadow-2xl transform scale-125 transition duration-500"
  >
    <h3 class="text-5xl font-extrabold text-white mb-4">組織的効率化</h3>
    <p class="text-2xl text-green-300 font-bold tracking-wide leading-relaxed">
      チーム間での<span class="text-yellow-300 font-bold">不要な調整が減る</span>ことで、<br />
      <span class="font-bold">開発全体が加速</span>する<br />
      セルフサービス化ができる
    </p>
  </div>
</div>

<!--
3つ目。  
以前は、実績データの仕様変更や、算定に必要なパラメータの仕様変更があった場合に密に連携が必要でした。又、双方の仕様が決まるまで繰り返し調整が必要でした。    
[click] インターフェースを定めることで、以降の開発は基本的に一方通行のコミュニケーションで完結し、調整にかかるコストが最低限で済む様になりました。      
[click] インターフェースとドキュメントの整備により、他チームとの不要な調整が減り、開発スピードも上がりました。「Rezept as a Service」をセルフサービス型のプラットフォームとして位置づけることができました。
-->

---
layout: center
class: text-center
---

# <carbon-train-speed /> 4. システム全体の開発アジリティ向上

<div mt-8 text-2xl>
  レセプト業務の「安定」は、<span class="font-bold">プロジェクト全体のスピード</span>に繋がる
</div>

<div class="w-full h-90 grid grid-cols-2 gap-8 mt-10">

  <div
    class="flex flex-col p-6 bg-gray-800/70 rounded-lg shadow-xl border-l-4 border-red-500 transition duration-500 ease-in-out"
  >
    <h3 class="text-3xl font-bold text-red-300 mb-4">🔴 Before (終盤に集中)</h3>
    <ul class="list-disc list-inside text-xl space-y-3 text-left">
      <li>多くのテストケースを<span class="font-bold">開発終盤</span>で一気に消化</li>
      <li><span class="font-bold">時間的プレッシャー</span>が非常に大きい</li>
      <li>見切り発車でによる並行開発で、手戻りが発生した場合の<span class="font-bold">コストが高い</span></li>
    </ul>
  </div>

  <div
    v-click="1"
    class="flex flex-col p-6 bg-gray-800/70 rounded-lg shadow-xl border-r-4 border-green-500 transition duration-500 ease-in-out"
    :class="$clicks < 1 ? 'opacity-0 scale-90' : 'opacity-100 scale-100'"
  >
    <h3 class="text-3xl font-bold text-green-300 mb-4">🟢 After (並行・早期実施)</h3>
    <ul class="list-disc list-inside text-xl space-y-3 text-left">
      <li>コアドメインの仕様を<span class="font-bold">先行確定</span></li>
      <li>各プロダクトチームが<span class="font-bold">並行開発</span></li>
      <li><span class="font-bold">早い段階から十分なテスト</span>を実施</li>
      <li>プロジェクト全体の<span class="font-bold">開発サイクルが大幅短縮</span></li>
    </ul>
  </div>
</div>

<div 
  v-click="2"
  class="absolute inset-0 z-20 flex items-center justify-center bg-black/50 backdrop-blur-sm transition duration-700 ease-in-out"
  :class="$clicks < 2 ? 'opacity-0 pointer-events-none' : 'opacity-100 pointer-events-auto'"
>
  <div 
    class="p-10 bg-white/10 backdrop-blur-lg border-4 border-green-400 rounded-2xl shadow-2xl transform scale-125 transition duration-500"
  >
    <h3 class="text-5xl font-extrabold text-white mb-4">アジリティの実現</h3>
    <p class="text-2xl text-green-300 font-bold tracking-wide leading-relaxed">
      <span class="text-yellow-300 font-bold">品質の確保</span>と<br />
      <span class="font-bold">開発効率の両立</span>を実現
    </p>
  </div>
</div>

<!--
4つ目。  
以前は、開発の終盤にテストが集中することが多くQAへの負担が大きかったです。又、見切り発車での並行開発で手戻りがコストも増加していました。    
[click] それがインターフェス仕様を先に決めることで、並行開発することができるようになりました。 
また独立してテストを行えたり、開発着手が早まる分、結合前に品質を確保することが可能になりました。   
[click] プロジェクト全体の開発サイクルが大幅に短縮され、品質にも寄与することができました。
-->

---
layout: center
class: text-center
zoom: 2.0
---

# <carbon-review /> 振り返り
2 つの Key Success Factor


1. 関心ごとの分離
2. 依存性逆転

<!--
ここまで、2つの成功要因について説明しました。
-->

---
layout: center
class: text-center
zoom: 2.0
---

# <mdi-confirmation-number /> なぜこの順番だったのか？ 🤔

<!--
ここで2つの成功要因には順番があるということに気づかれましたでしょうか？  
なぜ、この順番だったのでしょうか？
-->

---
layout: default
---

# <carbon-query /> なぜ関心ごとの分離が必要だったのか？
依存性逆転の前に、まず関心ごとの分離が必要

<v-clicks>

1. 認知サイズが大きすぎる  
   → 人間が一度に扱える複雑さには限界がある。全体を把握できない
   
2. 境界が不明確  
   → どこでインターフェスを切るべきか分からない。分離により、何が「上位」で何が「下位」かが見える
   
3. 関心ごとが混在している  
   → インターフェースが複雑になる。単一責務の原則に反し、リスクも大きくなる  

</v-clicks>

<v-click at="4">

<div class="speech-bubble">
まず認知サイズを削減してから、<br />
次のステップへ進む

これが現実的なアプローチ
</div>

</v-click>

<!--
依存性逆転の前に、まず「関心ごとの分離」を行いました。  
これは依存性逆転の難易度が高いということに起因します。  
[click] まず、当時のシステムは、認知サイズが大きすぎて把握できませんでした。　   
[click] 境界が不明確だと、依存関係も把握するのが難しいです。  
[click] 複数の関心ごとが混在していると、最初からインターフェースを切ろうとしたら、複雑になりすぎて使えない物になるでしょう。  
[click] まず関心ごとを分離して認知サイズを削減し、依存関係も明確化するのが良いと考えます。
-->

---
zoom: 0.8
---

# <carbon-shape-except /> 初期状態

<div class="text-red-600 text-xl">複雑性が混在（密結合）</div>

<div class="relative w-full h-100 flex items-center justify-center">
  <div 
    v-motion 
    :initial="{ scale: 0.2 }" 
    :enter="{ scale: 1.0 }" 
    class="absolute w-80 h-80 bg-yellow-400 rounded-full flex items-center justify-center text-center text-xl font-bold transition-all duration-700"
    style="z-index: 10;"
  >
    <span v-if="$clicks < 1">モジュールX</span>
    <span v-else><br /><br /><br /><br />モジュールX<br>(大きな泥団子状態)</span> 
  </div>

  <div 
    v-click="1" 
    v-motion 
    :initial="{ x: -300, y: -200, opacity: 0 }" 
    :enter="{ x: 0, y: 0, opacity: 60 }" 
    absolute 
    class="w-32 h-32 left-83 top-8 bg-blue-300 opacity-60 rounded-full flex items-center justify-center text-lg text-blue-900 transition-all duration-700" 
    style="z-index: 15;"
  >
    関心ごとA
  </div>

  <div 
    v-click="1" 
    v-motion 
    :initial="{ x: 300, y: -200, opacity: 0 }" 
    :enter="{ x: 0, y: 0, opacity: 60 }" 
    absolute 
    class="w-48 h-48 right-110 top-8 bg-green-300 opacity-60 rounded-full flex items-center justify-center text-lg text-green-900 transition-all duration-700"
    style="z-index: 15;"
  >
    関心ごとB
  </div>

  <div 
    v-click="1" 
    v-motion 
    :initial="{ x: -300, y: 200, opacity: 0 }" 
    :enter="{ x: 0, y: 0, opacity: 60 }" 
    absolute 
    class="w-48 h-48 left-70 bottom-2 bg-orange-300 opacity-60 rounded-full flex items-center justify-center text-lg text-orange-900 transition-all duration-700"
    style="z-index: 15;"
  >
    関心ごとD
  </div>

  <div 
    v-click="1" 
    v-motion 
    :initial="{ x: 300, y: 200, opacity: 0 }" 
    :enter="{ x: 0, y: 0, opacity: 60 }" 
    absolute 
    class="w-40 h-40 right-63 bottom-8 bg-purple-300 opacity-60 rounded-full flex items-center justify-center text-lg text-purple-900 transition-all duration-700"
    style="z-index: 15;"
  >
    関心ごとC
  </div>
</div>

<div class="mt-8 text-center text-red-600 text-xl" v-click="1">
  重なり: 最大<br>
  結合度: 非常に高<br>
  境界: 曖昧<br>
</div>
<div class="mt-2 text-center text-red-700 text-xl" v-click="1">
  認知サイズ: 非常に大
</div>

<!--
次のスライドから、ドメイン蒸留の流れを説明していきます。
初期フェーズではどんどん機能が追加されていき [click]「モジュールX」は「大きな泥団子」状態になっていきます。  
いっぱい関心ごとが重なり、結合度も高く、責任の境界も曖昧になります。  
雪だるま式にシステムのコントロールが難しくなっていきます。
-->

---
zoom: 0.8
---

# <carbon-bring-forward /> 方法1: 関心ごとの抽出

<div class="text-green-600 text-xl">Core内から別の関心ごとを分割</div>

<div class="relative w-full h-96 flex items-center justify-center">
  <div 
    v-click="1" 
    v-motion 
    :leave="{ scale: 1, y: 0 }" 
    :enter="{ scale: 1, y: 0 }" 
    absolute 
    :class="$clicks < 1? 'w-80 h-80 bg-yellow-400/80' : 'w-65 h-65 bg-yellow-400'" 
    class="rounded-full flex items-center justify-center text-center text-xl font-bold transition-all duration-700" 
    style="z-index: 10;"
  >
    <span v-if="$clicks < 1" class="py-12">Core<br>(巨大)</span>
    <span v-else class="py-12">Core<br>(縮小)</span>
  </div>

  <div class="absolute w-32 h-32 left-83 top-8 bg-blue-300 opacity-60 rounded-full flex items-center justify-center text-lg text-blue-900">
    関心ごとA
  </div>

  <div v-motion 
    absolute 
    :class="$clicks < 1 ? 'right-110 top-8 bg-green-400/60' : 'right-65 top--5 bg-transparent border-3 border-green-700 border-dashed'" 
    class="w-48 h-48 rounded-full flex items-center justify-center text-lg text-green-900 transition-all duration-700"
    style="z-index: 1;">
    <span v-if="$clicks < 1">関心ごとB</span>
    <span v-else>抽出<br>B</span>
  </div>
  
  <div class="absolute w-48 h-48 left-70 bottom-2 bg-orange-300 opacity-60 rounded-full flex items-center justify-center text-lg text-orange-900">
    関心ごとD
  </div>

  <div class="absolute w-40 h-40 right-63 bottom-8 bg-purple-300 opacity-60 rounded-full flex items-center justify-center text-lg text-purple-900">
    関心ごとC
  </div>

  <Arrow 
    v-click="1" 
    x1="600" y1="180" 
    x2="700" y2="100" 
    color="#38A169" 
    stroke-width="4" 
    head-size="10"
    :z-index="30"
  />
</div>

<div class="mt-8 text-center text-green-600 text-xl" v-click="1">
  重なり: 縮小<br>
  結合度: 改善<br>
  境界: 少し明確化<br>
</div>
<div class="mt-2 text-center text-blue-700 text-xl" v-click="1">
  認知サイズ: 中
</div>

<!--
そこで関心ごとを抽出するという取り組みが重要になります。  
[click] 先ほど紹介した時系列で関心ごとを分割するというのはこちらに当たります。  
こうすることで、重なりも減り、責務も明確になっていき、認知負荷もいい感じに改善されてきます。
-->

---
zoom: 0.8
---

# <carbon-send-to-back /> 方法2: 処理の移譲

<div class="text-blue-600 text-xl">重なり部分を他に移す</div>

<div class="relative w-full h-96 flex items-center justify-center">
  <div left-105 w-65 h-65 absolute bg-yellow-400 rounded-full flex items-center justify-center text-center text-xl font-bold>
    <span v-if="$clicks < 1">Core</span>
    <span v-else>Core<br>(最小)</span>
  </div>

  <div v-motion 
    absolute 
    :class="$clicks < 1? 'left-83 top-8' : 'left-81 top-6'" 
    style="z-index: 1;" class="absolute w-32 h-32 bg-blue-300 opacity-60 rounded-full flex items-center justify-center text-lg text-blue-900 transition-all duration-700">
    関心ごとA
  </div>

  <Arrow 
    v-click="1" 
    x1="445" y1="125" 
    x2="420" y2="110" 
    color="#1461e7ff" 
    stroke-width="4" 
    head-size="10"
    :z-index="30"
  />

  <div right-66 top-0 absolute w-48 h-48 bg-green-300 opacity-60 rounded-full flex items-center justify-center text-lg text-green-900>
    関心ごとB  
  </div>

  <div class="absolute w-48 h-48 left-70 bottom-2 bg-orange-300 opacity-60 rounded-full flex items-center justify-center text-lg text-orange-900">
    関心ごとD
  </div>

  <div class="absolute w-40 h-40 right-63 bottom-8 bg-purple-300 opacity-60 rounded-full flex items-center justify-center text-lg text-purple-900">
    関心ごとC
  </div>
</div>

<div class="mt-8 text-center text-blue-600 text-xl" v-click="1">
  重なり: 小<br>
  結合度: 低<br>
  境界: 明確化<br>
</div>
<div class="mt-2 text-center text-blue-700 text-xl" v-click="1">
  認知サイズ: 小
</div>

<!--
更に責務を移譲することも大事です。  
[click] 先に紹介した「法改正の影響がない周辺の関心ごとを外に出す」という取り組みがこれに当たります。  
関心ごとが切り離されて、責務が明確になります。
-->

---
zoom: 0.8
---

# <carbon-connect-recursive /> 依存性逆転 (DIP)

<div class="text-amber-700 text-xl">重なりをインターフェース境界として分離</div>

<div class="relative w-full h-96 flex items-center justify-center">
  <div left-105 w-65 h-65 absolute bg-yellow-400 rounded-full flex items-center justify-center text-center text-xl font-bold>
    <span v-if="$clicks < 1">Core</span>
    <span v-else>Core<br>(Pure)</span>
  </div>

  <div class="absolute w-32 h-32 left-81 top-6 bg-blue-300 opacity-60 rounded-full flex items-center justify-center text-lg text-blue-900">
    関心ごとA
  </div>

  <div right-66 top-0 absolute w-48 h-48 bg-green-300 opacity-60 rounded-full flex items-center justify-center text-lg text-green-900>
    関心ごとB  
  </div>

  <div v-motion 
    absolute 
    :class="$clicks < 1? 'left-70 bottom-2' : 'left-62 bottom-0'" 
    style="z-index: -1;" class="w-48 h-48 bg-orange-300 opacity-60 rounded-full flex items-center justify-center text-lg text-orange-900 transition-all duration-700">
    関心ごとD
  </div>

  <div class="absolute w-40 h-40 right-63 bottom-8 bg-purple-300 opacity-60 rounded-full flex items-center justify-center text-lg text-purple-900">
    関心ごとC
  </div>

  <div v-click="1" class="absolute w-6 h-6 left-108 bottom-32 bg-purple-600 rounded-full transition-all duration-700" style="z-index: 11;"></div>

  <Arrow v-click="1" x1="380" y1="280" x2="435" y2="250" color="#5b21b6" stroke-width="4" head-size="10" :z-index="30">
    <text ont-size="16" fill="#4308a3ff" text-anchor="middle">I/F D</text>
  </Arrow>
</div>

<div v-click="1" class="mt-8 text-center text-red-600 text-xl">
  <span class="text-fuchsia-700">重なり → インターフェースに集約</span><br>
  <span class="text-fuchsia-700">全ての関心ごとがI/Fに依存</span>
</div>
<div v-click="1" class="mt-2 text-center text-green-700 text-xl">
  結合度: 最小 (安定)<br>
  並行開発/交換容易
</div>

<!--
最後に「依存性逆転」になります。  
関心ごとの分離や、責務の移譲を行なったとしても、どうしても重なりが残ってしまうケースがあります。  
それは処理やデータフローの関係により、依存が残ってしまう部分です。  
[click]   依存性逆転の原則を適用し、共有部分をインターフェース化することで問題を解決できます。  
これにより、関心ごとが跨る部分をきれいに分離して、疎結合にすることができます。  
コアドメインは、結合度を最小限に抑え、外部に依存しないことで、最も安定した状態を実現することを目指します。
-->

---
layout: center
class: text-center
zoom: 2.0
---

# <carbon-star-review /> もっと重要なこと

<!--
そして、もっと重要なことがあります。
-->

---
layout: center
class: text-center
zoom: 2.0
---

# <carbon-radar /> コアドメインの見極め

<!--
それは「コアドメインの見極め」です。
-->

---
layout: center
class: text-center
zoom: 2.0
---

# <carbon-cube /> コアドメインとは？


<blockquote>
製品やサービスで最も重要で、競争優位性の源泉となる中核的な業務領域です。ビジネス上の差別化を図る部分
</blockquote>

<!--
コアドメインとは、ビジネスの競争優位の源泉となる中核的な領域です。
-->

---
layout: center
class: text-center
zoom: 2.5
---

# 🤔 よくわからん

<!--
抽象度が高く、理解しづらく調べてもよくわからない何かです。    
そんなコアドメインですが、どの様に見つければ良いのでしょうか？
-->

---

# <carbon-diamond-outline /> ダブルダイヤモンド
デザイン思考の「問題解決フレームワーク」

<div max-w-160>

<img src="./public/double_diamond.svg" />

</div>

<!--
説明の為に、デザイン思考のフレームワークを拝借します。ダブルダイヤモンドというものです。  
ダブルダイヤモンドでは問題発見（探索と定義）と解決策の創出（開発と実現）の2つのフェーズが存在します。
-->

---
layout: center
class: text-center
zoom: 2.0
---

# <carbon-incident-reporter /> 問題領域に向き合う必要がある
左のダイヤモンド。解くべき問題を定める

<!--
コアドメインは、この問題領域に存在しています。  
解くべき問題を正しく認識することが重要になります。
-->

---
layout: center
class: text-center
zoom: 2.0
glow: full
glowSeed: 300
---

# <carbon-strategy-play /> 問題領域の定義したものは戦略となる
逆に解決領域の提供が戦術となる。戦略を実現する為の戦術という位置づけ

<!--
重要なのは、ビジネスが扱っている問題に対して、どう解くか（戦術）の前に、「何を解くべきか」（戦略）を定義することです。  
戦略に紐づかない戦術は駄目なパターンです。  
「それはhowなんよ〜」という声が聞こえそうです。  
事業に向き合って、戦略を立てて、それに投資するという判断が必要です。
-->

---
layout: default
zoom: 1.1
---

# <carbon-task-view /> 今回の問題の洗い出し

* 法改正のたびに大きく変わるが、旧制度のルールは壊してはいけない
* 法令知識を持ってないとメンテナンスができない
  * 認知負荷が高すぎる。開発者全員が深い法令知識があるわけではない
  * 法令に関する影響範囲が分散してしまうと、法改正時の修正漏れが発生する
* 請求業務のテストパターンが膨大だが、構造的に後回しになってしまう
* 法改正のシステム対応が間に合わないと、請求ができなくなる  
間違って請求した場合のインパクトも大
<div v-click="1" class="mx-auto w-full max-w-4xl transition duration-500 ease-in-out"
  :class="$clicks < 1 ? 'opacity-0 scale-90' : 'opacity-100 scale-100'">
  
<div class="p-6 bg-black/50 border-2 border-yellow-400 rounded-lg shadow-xl">
    
  <h3 class="text-2xl font-extrabold text-white mb-4">🔑 コアドメインの判断軸</h3>
    
  <div class="grid grid-cols-2 gap-4 text-left">
      
  <div class="text-xl space-y-2">
    <div class="font-bold">1. 複雑で、頻繁に変更される</div>
      <div class="font-bold">2. 知識が凝縮され、暗黙知が多い</div>
    </div>
      
  <div class="text-xl space-y-2">
    <div class="font-bold">3. 壊れた時の影響が大きい</div>
      <div class="font-bold">4. 分散させると問題になる</div>
    </div>
  </div>
</div>
</div>

<!--
今回私たちが洗い出した「解くべき問題」は、「法改正で頻繁に変わる」「法令知識がないと触れない」「認知負荷が高すぎる」「影響範囲が分散すると修正漏れが起きる」といったものでした。  
今回は、競争優位性の為の問題ではなく、事業リスクの観点を中心とした問題となりました。  
こちらの問題は今回のケースで探索された内容で、一概にコアドメインを定めるものではありません。  
[click] ただ問題を抽象化して考えてみると、こちらの4つが、コアドメインの悲観的な判断軸にできると個人的に考えています。
-->

---
zoom: 1.4
---

# <carbon-strategy-play /> 解くべき問題に対する戦略

* 法改正毎の算定ルールを分けて管理する  
全制度を知っていなくても開発ができる。  
旧制度に影響がない状態にする `(開放/閉鎖の原則)`
* レセプト業務の関心ごとを凝集して管理する  
複雑さをシステム全体に拡散させない
* レセプト業務の知識（算定基準）を専門性を持つチームに集中させる  
他チームは法令知識の内、運営基準・体制基準にフォーカスさせる

<!--
先ほどの問題に対する「戦略」を改めて言語化してみました。  
「算定ルールを分けて管理する」「レセプト業務の関心ごとを凝集する」「専門知識を特定チームに集中させる」という内容です。  
この内容は戦略そのものです。
-->

---
layout: center
class: text-center
transition: slide-up
---

# <carbon-directory-domain /> 今回のコアドメインの特定

<div mt-8 text-2xl>
  課題分析に基づき、最も投資すべき<span class="font-bold text-yellow-300">「コアドメイン」</span>を決定
</div>

<div class="w-full max-w-4xl m-auto mt-16 p-10 bg-gray-800/80 border-4 border-yellow-400 rounded-2xl shadow-2xl transition duration-500 ease-in-out transform hover:scale-105">
  
  <h3 class="text-4xl font-extrabold text-white mb-6 flex items-center justify-center gap-4">
    <div i-carbon:drill-up text-5xl text-yellow-400 />コアドメインとして設定するのは...
  </h3>
  
  <div class="p-5 bg-black/30 rounded-lg border border-yellow-500">
    <p class="text-3xl font-bold text-yellow-300">
      レセプト業務をコアドメインとして、<br />
      <span class="text-white text-4xl">Rezept as a Service</span> を構築する
    </p>
  </div>
  
</div>

<div 
  v-click="1"
  class="absolute bottom-10 left-0 right-0 z-20 transition duration-700 ease-in-out"
  :class="$clicks < 1 ? 'opacity-0 translate-y-10' : 'opacity-100 translate-y-0'"
>
  <div 
    class="p-4 bg-red-800/80 backdrop-blur-sm border-2 border-red-500 rounded-lg shadow-xl inline-block"
  >
    <p class="text-2xl font-bold text-white tracking-wide">
      システムの安定性とアジリティを確保するための<br />
      <span class="text-red-300 text-3xl">【戦略的な投資判断】</span>
    </p>
  </div>
</div>

<!--
この戦略に対して集中すべき関心ごとが「コアドメイン」となります。  
今回はレセプト業務をコアドメインとして、戦略に基づきRezept as a Serviceを構築する流れとなり [click] 専用チームも作るという投資判断も行いました。
-->

---

# <carbon-tools /> 戦略に対する戦術
２回のリアーキテクチャの取り組みが戦術となる

* 時系列で API をルーティングさせる
* スキーマ駆動開発

<v-click>

<div class="speech-bubble">
これらも戦術<br />
* APIシナリオテスト（runn）<br />
* バージョン管理・リリースフローの整備（tagpr）<br />
* APIインターフェースの品質向上（eg-r2）<br />
</div>

<OgpImage url="https://zenn.dev/litalico/articles/re-architecting-rezept2" />

</v-click>

<!--
この戦略を実現するための「戦術」が、これまでお話ししてきた2回のリアーキテクチャだったわけです。  
[click] 時間の都合で細かく説明できませんが、スキーマ駆動開発は戦略を実現する為の戦術になります。   
runnによるAPIシナリオテストや、tagprによるバージョン管理・リリースフローの整備、eg-r2によるAPIインターフェースの品質向上などは戦術です。    
こちらも別途記事でまとめていますので、ぜひご参照ください。
-->

---
layout: center
class: text-center
zoom: 3.0
glow: right
---

# <carbon-document-tasks /> まとめ

<!--
まとめです。
-->

---
glowSeed: 49
---

# <carbon-cube /> コアドメインの本質的な役割
コアドメインは戦略によって選択される

複雑で変わりやすいドメインロジックを「一箇所に集約」して「隔離」する

目的:
- 影響範囲を局所化
- 認知負荷を特定のチームに集中
- サブドメインチームはコアドメインの詳細を知らなくても開発できる状態

<!--
コアドメインの本質的な役割は、複雑で変わりやすいロジックを「一箇所に集約」し、システム全体から「隔離」することです。  
コアドメインは戦略によって選択され、一番安定させる取り組みを行います。
-->

---
glowSeed: 50
---

# <carbon-badge /> 依存性逆転の真の価値
インターフェースは『複雑さからの防壁』

コアドメインチーム:
- コアドメインに集中できる
- 他システムの実装詳細から切り離される
- テスト範囲が明確

サブドメインチーム:
- コアドメインの詳細を知らなくていい
- インターフェース仕様だけ理解すればいい
- 自分のドメインに集中できる

<!--
「依存性逆転」の真の価値は、インターフェースを「複雑さからの防壁」として機能させることです。  
これにより、コアドメインチームもサブドメインチームも、お互いの詳細を知らなくても、自身のドメイン開発に集中できる状態を作り出せました。
-->

---
layout: center
class: text-center
transition: slide-up
glowSeed: 51
---

# <carbon-education /> 3つの重要な教訓 ✨

<div class="w-full grid grid-cols-3 gap-8 m-auto mt-12 max-w-4xl">

  <div v-click="1" class="border-4 border-red-600 bg-red-600/20 rounded-xl p-5 transition duration-500 ease-in-out"
       :class="$clicks < 1 ? 'opacity-0 translate-y-10' : 'opacity-100 translate-y-0'">
    <div i-carbon:location-star text-5xl text-yellow-400 mb-3 />
    <span class="font-bold text-xl text-white"><i-carbon:location-star /> 1. 複雑性は隔離</span>
    <hr class="my-3 border-red-500">
    <ul class="list-none text-base space-y-2 text-left">
      <li>最も複雑で壊れやすい部分を<span class="font-bold">一箇所に集約</span></li>
      <li>分散させると<span class="font-bold">問題が拡大</span></li>
      <li><span class="font-bold">認知負荷</span>の適切な分配</li>
    </ul>
  </div>

  <div v-click="2" class="border-4 border-yellow-600 bg-yellow-600/20 rounded-xl p-5 transition duration-500 ease-in-out"
       :class="$clicks < 2 ? 'opacity-0 translate-y-10' : 'opacity-100 translate-y-0'">
    <div i-carbon:rule-partial text-5xl text-yellow-400 mb-3 />
    <span class="font-bold text-xl text-white">2. 境界を明確化</span>
    <hr class="my-3 border-yellow-500">
    <ul class="list-none text-base space-y-2 text-left">
      <li>関心ごとの違いを<span class="font-bold">認識</span></li>
      <li>クロスポイントを<span class="font-bold">インターフェースとして契約化</span></li>
      <li><span class="font-bold">詳細を知らない状態</span>（疎結合）の実現</li>
    </ul>
  </div>

  <div v-click="3" class="border-4 border-green-600 bg-green-600/20 rounded-xl p-5 transition duration-500 ease-in-out"
       :class="$clicks < 3 ? 'opacity-0 translate-y-10' : 'opacity-100 translate-y-0'">
    <div i-carbon:data-refinery text-5xl text-green-400 mb-3 />
    <span class="font-bold text-xl text-white">3. 依存の方向を制御</span>
    <hr class="my-3 border-green-500">
    <ul class="list-none text-base space-y-2 text-left">
      <li>データの流れ ≠ <span class="font-bold">依存の方向</span></li>
      <li>重要な部分を<span class="font-bold">安定した位置</span>に配置</li>
      <li><span class="font-bold">並行開発</span>を可能にする構造</li>
    </ul>
  </div>

</div>

<!--
本日のまとめとして、3つの重要な教訓をお伝えします。  
[click] 1つ目、「複雑性は隔離」すること。最も複雑で壊れやすい部分は一箇所に集約します。  
[click] 2つ目、「境界を明確化」すること。関心ごとの違いを認識し、インターフェースとして契約化します。  
[click] 3つ目、「依存の方向を制御」すること。データの流れと依存の方向を同一視せず、最も重要な部分を最も安定した位置に配置します。
-->

---
glowSeed: 52
---

# <carbon-link /> 参考URL
時系列で記事をまとめています

* [法改正をマイクロサービスで立ち向かう（前編）](https://zenn.dev/litalico/articles/confronting-law-amends-with-microservices)
* [法改正をマイクロサービスで立ち向かう（後編）](https://zenn.dev/litalico/articles/confronting-law-amends-with-microservices2)
* [ビジネス変化に適用する為にリアーキテクティングを行うまでの話](https://zenn.dev/litalico/articles/re-architecting-rezept)
* [スキーマ駆動開発を支える開発プロセス＆ツール群](https://zenn.dev/litalico/articles/re-architecting-rezept2)
* [アーキテクチャレベルで依存性を逆転させたら最高だった話](https://zenn.dev/litalico/articles/domain-distillation)

<!--
関連する記事を時系列でまとめていますので、よろしければご覧ください。
-->

---
layout: end
zoom: 2.0
---

ご清聴ありがとうございました

<!--
ご清聴ありがとうございました。
-->

---
layout: center
class: text-center
transition: fade
---

# <icon-park-outline-pigeon /> あなたの技術が、社会を変える 🌎✨

<div mt-10 text-3xl font-bold>
  <span class="text-yellow-300">教育・福祉領域×Tech</span> で、<br />
  <span class="text-green-300">「障害のない社会をつくる」</span>挑戦をしませんか？
</div>

<div class="flex justify-center items-center gap-12 mt-16">
  
  <div class="flex flex-col items-center">
    <h3 class="text-3xl font-extrabold text-white mb-4">
      🚀 募集要項はこちら
    </h3>
    <div class="p-4 bg-white rounded-lg shadow-2xl">
      <QRCode text="https://herp.careers/v1/litalico/requisition-groups/ea1f6b9d-88b7-4254-a06a-8cfdd83bd575" class="w-36 h-36" />
    </div>
    <a href="https://herp.careers/v1/litalico/requisition-groups/ea1f6b9d-88b7-4254-a06a-8cfdd83bd575" target="_blank" class="text-xl mt-4 font-bold text-blue-400 hover:text-blue-300">
      詳細を見る（HERP）
    </a>
  </div>

  <div class="flex flex-col items-center border-l-2 border-gray-600 pl-12">
    <h3 class="text-3xl font-extrabold text-white mb-4">
      🗣️ 登壇者へ話しかける
    </h3>
    <div class="p-6 bg-yellow-800/20 border-2 border-yellow-500 rounded-lg shadow-lg">
      <p class="text-xl font-bold text-white">
        ご興味のある方は、ぜひ<br />
        <span class="text-yellow-300 text-4xl">katzumi</span> まで<br />
        お気軽にお声がけください！
      </p>
    </div>
  </div>

</div>

<!--
最後に宣伝させてください。  
私たちは「障害のない社会をつくる」チャレンジをしています。ご興味のある方は、ぜひお気軽にお声がけください！
-->
