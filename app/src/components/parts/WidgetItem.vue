<template>
  <div class="widget-family">
    <div
      class="widget"
      @mouseover="onMouseOver"
      @mouseleave="onMouseLeave"
      v-bind:class="{ mouseover: widget.mouseover }"
    >
      <!-- テキスト入力欄 -->
      <template v-if="widget.type == 'heading'">
        <input
          v-model="widget.text"
          class="heading transparent"
          placeholder="見出し"
          v-bind:ref="'widget-heading-' + widget.id"
        />
      </template>
      <template v-if="widget.type == 'body'">
        ・<input
          v-model="widget.text"
          class="body transparent"
          placeholder="本文"
          v-bind:ref="'widget-body-' + widget.id"
        />
      </template>
      <template v-if="widget.type == 'code'">
        <textarea
          v-model="widget.text"
          class="code"
          rows="1"
          placeholder="コード"
          v-bind:ref="'widget-code-' + widget.id"
        >
        </textarea>
      </template>

      <!-- ボタン群 -->
      <div v-show="widget.mouseover" class="buttons">
        <div
          class="button-icon"
          v-if="layer < 3"
          @click="onClickChildWidget(widget)"
        >
          <i class="fas fa-sitemap"></i>
        </div>
        <div
          class="button-icon"
          @click="onClickAddWidgetAfter(parentWidget, widget)"
        >
          <i class="fas fa-plus-circle"></i>
        </div>
        <div class="button-icon" @click="onClickDelete(parentWidget, widget)">
          <i class="fas fa-trash"></i>
        </div>
        <div class="button-icon">
          <i class="fas fa-cog" data-toggle="dropdown"></i>
          <div class="dropdown-menu">
            <a class="dropdown-item" @click="widget.type = 'heading'">見出し</a>
            <a class="dropdown-item" @click="widget.type = 'body'">本文</a>
            <a class="dropdown-item" @click="widget.type = 'code'"
              >ソースコード</a
            >
          </div>
        </div>
      </div>
    </div>

    <div class="child-widget">
      <draggable v-bind:list="widget.children" group="widgets">
        <WidgetItem
          v-for="childWidget in widget.children"
          v-bind:widget="childWidget"
          v-bind:parentWidget="widget"
          v-bind:layer="layer + 1"
          v-bind:key="childWidget.id"
          @delete="onClickDelete"
          @addChild="onClickChildWidget"
          @addWidgetAfter="onClickAddWidgetAfter"
        />
      </draggable>
    </div>
  </div>
</template>

<script>
import draggable from "vuedraggable";

export default {
  name: "WidgetItem",
  props: ["widget", "parentWidget", "layer"],
  /* ライフサイクルとライフサイクルフック
  
  Vue.jsを始めとするフレームワークには、表示されている要素が作成、更新、削除されるまでの一連の流れである、
  「ライフサイクル」という概念があります。
  
  その流れの中で、特定のタイミングで予め決められた関数が呼び出されるような仕組みが用意されています。
  これを「ライフサイクルフック」と呼びます。
  例えばデータの初期化が完了したらcreated関数、データが更新されたらupdated関数が呼び出される、といった形です。
  
  今回使用しているmountedは、初期化後にHTML部分の準備ができたタイミングで呼び出されるものなので、
  このタイミングで入力要素にフォーカスを当てるような処理を行うことで
  作成された要素がすぐに入力できる状態を実現することができます。
  */
  mounted: function () {
    const input = this.$refs[`widget-${this.widget.type}-${this.widget.id}`];
    input.focus();
  },

  methods: {
    onMouseOver: function () {
      this.widget.mouseover = true;
    },
    onMouseLeave: function () {
      this.widget.mouseover = false;
    },
    onClickDelete: function (parentWidget, widget) {
      this.$emit("delete", parentWidget, widget);
    },
    onClickChildWidget: function (widget) {
      this.$emit("addChild", widget);
    },
    onClickAddWidgetAfter: function (parentWidget, widget) {
      this.$emit("addWidgetAfter", parentWidget, widget);
    },
    // ソースコードタイプの高さを設定
    resizeCodeTextarea: function () {
      if (this.widget.type !== "code") return;
      const textarea = this.$refs[`widget-code-${this.widget.id}`];
      const promise = new Promise(function (resolve) {
        resolve((textarea.style.height = "auto"));
      });
      promise.then(function () {
        // スクロールバー全体の長さをtextareaの高さとして設定
        textarea.style.height = textarea.scrollHeight + "px";
      });
    },
  },
  /*
  watchオプション
  watchオプションは指定した変数の変化を検知して、任意の処理を行う仕組みとなります。
  
  今回は使用していませんが、変更前の値と変更後の値をそれぞれ引数として受け取ることもでき、
  それらの値を条件として処理を分岐することも可能となります。
  
  値の変化に応じて単純な計算結果を返すような場合はcomputedが用いられますが、
  今回のように計算結果を返すだけではなく複雑な処理が必要な場面ではwatchが必要となります。
  */
  watch: {
    "widget.text": function () {
      this.resizeCodeTextarea();
    },
  },
  components: {
    draggable,
  },
};
</script>

<style scoped lang="scss">
.widget {
  width: 100%;
  min-height: 40px;
  display: flex;
  align-items: center;
  padding: 5px 10px;
  color: rgba(25, 23, 17, 0.6);
  .buttons {
    display: flex;
    flex-direction: row;
    cursor: pointer;
    .button-icon {
      padding: 3px;
      margin-left: 3px;
      border-radius: 5px;
    }
  }
  input.heading {
    font-size: 20px;
    font-weight: bold;
    border-bottom: 1.5px solid #e0e0e0;
  }
  input.body {
    font-size: 16px;
  }
  .code {
    width: calc(100% - 120px);
    height: 35px;
    padding: 5px 10px;
    border: none;
    border-radius: 8px;
    color: #f8f8f2;
    background: #282a36;
    font-size: 14px;
    font-family: Consolas, Menlo, "Liberation Mono", Courier, monospace;
    resize: none;
  }
  .code:focus {
    outline: none;
    -webkit-box-shadow: none;
    box-shadow: none;
  }
}
.child-widget {
  padding-left: 10px;
}
</style>
