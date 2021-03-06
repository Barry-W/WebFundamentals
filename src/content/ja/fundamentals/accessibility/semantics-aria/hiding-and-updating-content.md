project_path: /web/_project.yaml
book_path: /web/fundamentals/_book.yaml
description: 支援技術に対してコンテンツを非表示にする


{# wf_updated_on: 2016-10-04 #}
{# wf_published_on: 2016-10-04 #}

# コンテンツの非表示と更新 {: .page-title }

{% include "web/_shared/contributors/megginkearney.html" %}
{% include "web/_shared/contributors/dgash.html" %}
{% include "web/_shared/contributors/aliceboxhall.html" %}

##  aria-hidden

支援技術を利用するユーザーのエクスペリエンスを最適化するうえで重要なテクニックがもう 1 つあります。それは、ページ内で関連性の高い部分のみを支援技術に提供することです。選択した DOM をアクセシビリティ API から参照できないようにする方法はいくつかあります。


まず、DOM に対して明示的に非表示の要素は、アクセシビリティ ツリーにも含まれません。したがって、CSS スタイルで `visibility:
hidden` または `display: none` が指定されている要素、および HTML5 の `hidden` 属性が使用されている要素も、支援技術ユーザーに対して非表示になります。


ただし、視覚的にレンダリングされていなくても、明示的に非表示になっていない要素は依然としてアクセシビリティ ツリーに含まれます。よく使われるのは、絶対位置で画面外に配置された要素に「スクリーン リーダー限定テキスト」を含める方法です。



    .sr-only {
      position: absolute;
      left: -10000px;
      width: 1px;
      height: 1px;
      overflow: hidden;
    }
    

また、以前に説明したとおり、`aria-label`、`aria-labelledby`、`aria-describedby` の各属性を使用してスクリーン リーダー限定テキストを指定することもできます。



「スクリーン リーダー限定」テキストの作成の詳細については、WebAIM の[テキストを非表示にする技術](http://webaim.org/techniques/css/invisiblecontent/#techniques){: .external }に関する記事をご覧ください。



最後に、ARIA では視覚的に非表示になっていないコンテンツを、支援技術から除外するための仕組みが提供されています。それには `aria-hidden` 属性を使用します。この属性を要素に適用すると、アクセシビリティ ツリーから効果的に要素と*その子孫*を削除できます。ただし、`aria-labelledby` または `aria-describedby` 属性によって参照される要素は唯一の例外です。


    <div class="deck">
      <div class="slide" aria-hidden="true">
        Sales Targets
      </div>
      <div class="slide">
        Quarterly Sales
      </div>
      <div class="slide" aria-hidden="true">
        Action Items
      </div>
    </div>

たとえば、メイン ページへのアクセスをブロックするモーダル UI を作成する場合、`aria-hidden` を使用することがあります。この場合、視覚に障がいのないユーザーは、ほとんどのページが現在使用できないことを示す半透明のオーバーレイなどを見ることができますが、スクリーン リーダーのユーザーはそのまま、ページ内のその他の部分にアクセスする可能性があります。また、[以前に説明した](/web/fundamentals/accessibility/focus/using-tabindex#modals-and-keyboard-traps)キーボードのトラップが発生する可能性があるため、ページで現在対象外になっている部分が `aria-hidden` に指定されていることも確認する必要があります。




これで、ARIA を使用してネイティブ HTML セマンティクスを調整する方法、アクセシビリティ ツリーに大幅な変更を加える方法、1 つの要素のセマンティクスを変更する方法など、ARIA の基本について理解できました。続いて、緊急性の高い情報を伝えるために ARIA を活用するための方法について説明します。




##  aria-live

`aria-live` を使用すると、ページの一部に「live」マークを付けることができます。これは、ユーザーがページ上で更新された場所にたまたまアクセスするまで待つのではなく、ページのどこを参照していても最新情報を即座にユーザーに伝える必要があることを意味します。要素に `aria-live` 属性が指定されていると、その要素を含むページの部分とその子孫は*ライブ領域*と呼ばれます。



![ARIA live でライブ領域を作成](imgs/aria-live.jpg)

たとえば、ユーザーのアクションの結果として表示されるステータス メッセージなどがライブ領域にあたります。視覚に障がいのないユーザーに注意を喚起するほど重要なメッセージであれば、`aria-live` 属性を設定して支援技術のユーザーにも注意を促すことが重要です。まず、単純な `div` は次のとおりです。


    <div class="status">Your message has been sent.</div>
    

"live" を指定した場合と比較してみましょう。


    <div class="status" aria-live="polite">Your message has been sent.</div>
    

`aria-live` には、`polite`、`assertive`、`off` の 3 つの値を使用できます。

 - `aria-live="polite"` は、現在実行している操作にかかわらず、その操作が完了した時点でこの変更についてユーザーに通知するように支援技術に指示します。重要かつ急を要さない情報に最適です。`aria-live` を使用する多くのケースでこの値を使用します。


 - `aria-live="assertive"` は、操作中の内容にかかわらず割り込み、この変更を即座にユーザーに通知するように支援技術に指示します。この値は、重要かつ急を要する更新にのみ使用します。たとえば、「サーバーエラーが発生しました。変更は保存されていません。ページを更新してください」というステータス メッセージを表示したり、ユーザーの操作を直接反映して入力フィールドを更新する場合などが該当します。




 - `aria-live="off"` は、`aria-live` の割り込みを一時的に中断するように支援技術に指示します。


ライブ領域を適切に機能させるには、いくつかコツがあります。

まず、`aria-live` 領域は、最初のページの読み込みで設定する必要があります。これは厳格なルールではありませんが、`aria-live` 領域について問題が発生した場合は、これが原因である可能性があります。



2 番目に、スクリーン リーダーはそれぞれ、変更の種類に応じて反応が異なります。たとえば、一部のスクリーン リーダーでは、子孫要素の `hidden` スタイルを true から false に切り替えるだけで、アラートをトリガーできます。


`aria-live` と連動するその他の属性を使えば、ライブ領域が変更されたときにユーザーに伝える内容を細かく調整できます。


`aria-atomic` は、更新を伝えるときに領域全体を 1 つのまとまりと見なすように指定します。たとえば、年、月、日で構成される日付ウィジェットで `aria-live=true` および `aria-atomic=true` を指定し、ユーザーがステッパー コントロールで月だけを変更した場合は、日付ウィジェットのコンテンツ全体がもう一度読み上げられます。`aria-atomic` の値には、`true` または `false`（デフォルト）を指定します。





`aria-relevant` には、ユーザーに伝える必要がある変更の種類を指定します。複数のオプションがあり、個別に、またはトークンリストとして使用できます。


 - *additions*: ライブ領域に追加するすべての要素に意味があることを意味します。たとえば、既存のステータス メッセージのログに span を追加すると、その span についてユーザーに知らせるということになります（`aria-atomic` は `false` とします）。


 - *text*: 子孫ノードに追加されるテキスト コンテンツに関連することを意味します。たとえば、カスタム テキスト フィールドの `textContent` プロパティを変更すると、変更されたテキストがユーザーに読み上げられます。

 - *removals*: テキストまたは子孫ノードの削除についてユーザーに伝える必要があることを意味します。

 - *all*: すべての変更に関連することを意味します。ただし、`aria-relevant` のデフォルト値は `additions text` です。つまり、`aria-relevant` を指定しない場合、要素に追加されたものはすべてユーザーに通知されます。これは多くの場合、目的に合致しています。




最後に、`aria-busy` を使用すると、たとえばなにかを読み込んでいるときなどに、一時的に要素への変更を無視するように支援技術に指示できます。すべてが配置されたら、`aria-busy` を false に設定し、リーダーを通常の動作に戻す必要があります。


 


{# wf_devsite_translation #}
