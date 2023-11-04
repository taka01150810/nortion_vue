<template>
  <div class="main-page">
    <div class="left-menu">
      <!-- ノートリスト -->
      <!-- v-bind = コンポーネントに値を受け渡しを行うディレクティブ -->
      <NoteItem
        v-for="note in noteList"
        v-bind:note="note"
        v-bind:key="note.id"
      />

      <!-- ノート追加ボタン -->
      <button class="transparent" @click="onClickButtonAdd">
        <i class="fas fa-plus-square"></i>ノートを追加
      </button>
    </div>
    <div class="right-view">右ビュー</div>
  </div>
</template>

<script>
import NoteItem from "@/components/parts/NoteItem.vue";

export default {
  data() {
    return {
      // 1. 変数の定義
      noteList: [],
    };
  },
  methods: {
    // 2. 関数の定義
    /*
    dataで定義した変数や、methodsで定義した関数をスクリプト側で使用したい場合、
    this.変数名、this.関数名()といったようにthis.を初めにつけることで呼び出すことができます。
    */
    onClickButtonAdd: function () {
      this.noteList.push({
        id: new Date().getTime().toString(16),
        name: `新規ノート`,
      });
    },
  },
  components: {
    NoteItem,
  },
};
</script>

<style scoped lang="scss">
.main-page {
  display: flex;
  height: calc(100vh - 60px);
  .left-menu {
    width: 350px;
    background-color: #f7f6f3;
  }
  .right-view {
    flex-grow: 1;
    padding: 10px;
  }
}
</style>
