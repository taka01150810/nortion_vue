<template>
  <div class="main-page">
    <div class="left-menu" @click.self="onEditNoteEnd()">
      <!-- ノートリスト -->
      <!-- v-bind = コンポーネントに値を受け渡しを行うディレクティブ -->
      <!-- @deleteイベントが発覚したタイミングでonDeleteNoteメソッドを呼び出す -->
      <NoteItem
        v-for="note in noteList"
        v-bind:note="note"
        v-bind:key="note.id"
        v-bind:layer="1"
        @delete="onDeleteNote"
        @editStart="onEditNoteStart"
        @editEnd="onEditNoteEnd"
        @addChild="onAddChildNote"
        @addNoteAfter="onAddNoteAfter"
      />

      <!-- ノート追加ボタン -->
      <button class="transparent" @click="onClickButtonAdd">
        <i class="fas fa-plus-square"></i>ノートを追加
      </button>
    </div>
    <!-- 
      @click="onEditNoteEnd()"とすると、編集ボタンをクリックしたイベントが親である左ビューまで伝わることで
      onEditNoteEndが即座に呼ばれてしまい、編集状態がすぐに終了してしまうというバグが発生する。
      .selfという箇所を追加することで、「子から伝わったイベントではなく、左ビュー自体がクリックされた場合のみ動作する」
      という仕組みが実現可能。
    -->
    <div class="right-view" @click.self="onEditNoteEnd()">右ビュー</div>
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
    // ノート追加
    onClickButtonAdd: function () {
      this.onAddNoteCommon(this.noteList);
    },
    onAddNoteCommon: function (targetList, layer, index) {
      layer = layer || 1;
      const note = {
        id: new Date().getTime().toString(16),
        name: `新規ノート-${layer}-${targetList.length}`,
        mouseover: false,
        editing: false,
        children: [],
        layer: layer,
      };
      if (index == null) {
        targetList.push(note);
      } else {
        targetList.splice(index + 1, 0, note);
      }
    },
    // 引数 deleteNoteはemitの第二引数の変数が入ってくる
    onDeleteNote: function (parentNote, note) {
      const targetList =
        parentNote == null ? this.noteList : parentNote.children;
      const index = targetList.indexOf(note);
      targetList.splice(index, 1);
    },
    onEditNoteStart: function (editNote, parentNote) {
      const targetList =
        parentNote == null ? this.noteList : parentNote.children;
      for (let note of targetList) {
        note.editing = note.id === editNote.id;
        // 子ノートを基準として再帰的に呼び出し
        this.onEditNoteStart(editNote, note);
      }
    },
    onEditNoteEnd: function (parentNote) {
      const targetList =
        parentNote == null ? this.noteList : parentNote.children;
      for (let note of targetList) {
        note.editing = false;
        this.onEditNoteEnd(note);
      }
    },
    // 子ノート追加
    onAddChildNote: function (note) {
      this.onAddNoteCommon(note.children, note.layer + 1);
    },
    onAddNoteAfter: function (parentNote, note) {
      const targetList =
        parentNote == null ? this.noteList : parentNote.children;
      const layer = parentNote == null ? 1 : note.layer;
      const index = targetList.indexOf(note);
      this.onAddNoteCommon(targetList, layer, index);
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
