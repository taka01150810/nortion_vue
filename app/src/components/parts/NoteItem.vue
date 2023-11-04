<template>
  <div
    class="note"
    @mouseover="onMouseOver"
    @mouseleave="onMouseLeave"
    v-bind:class="{ mouseover: note.mouseover && !note.editing }"
  >
    <!-- 
        <template>タグは「vueコード上は存在するが、描画時には消えてしまう」という特徴を持った要素
        <template>タグの中で条件付けをすることで、無駄な要素を増やすことなく表示の切り替えを実現することができる
    -->
    <!-- 
        @keypressは キー入力イベント 
    -->
    <template v-if="note.editing">
      <input
        v-model="note.name"
        class="transparent"
        @keypress.enter="onEditEnd"
      />
    </template>
    <template v-else>
      <div class="note-icon">
        <i class="fas fa-file-alt"></i>
      </div>
      <div class="note-name">{{ note.name }}</div>

      <div v-show="note.mouseover" class="buttons">
        <div class="button-icon" @click="onClickEdit(note)">
          <i class="fas fa-sitemap"></i>
        </div>
        <div class="button-icon">
          <i class="fas fa-plus-circle"></i>
        </div>
        <div class="button-icon">
          <i class="fas fa-edit"></i>
        </div>
        <!-- ボタン押下時にイベントを発火。引数としてnoteを渡す。-->
        <div class="button-icon" @click="onClickDelete(note)">
          <i class="fas fa-trash"></i>
        </div>
      </div>
    </template>
  </div>
</template>

<script>
export default {
  name: "NoteItem",
  props: ["note"],
  methods: {
    onMouseOver: function () {
      this.note.mouseover = true;
    },
    onMouseLeave: function () {
      this.note.mouseover = false;
    },
    /*
    削除ボタンが押されたことを、呼び出し元のメインページ側に伝える必要がある。
    呼び出し元ではタグ内で@deleteというディレクティブでイベントを受け取り、
    そこから任意の処理へとつなげていくことができる。
    */
    onClickDelete: function (note) {
      this.$emit("delete", note);
    },
    onClickEdit: function (note) {
      this.$emit("editStart", note);
    },
    onEditEnd: function () {
      this.$emit("editEnd");
    },
  },
};
</script>

<style scoped lang="scss">
.note {
  margin: 10px 0;
  display: flex;
  align-items: center;
  padding: 5px;
  color: rgba(25, 23, 17, 0.6);
  &.mouseover {
    background-color: rgb(232, 231, 228);
    cursor: pointer;
  }
  .note-icon {
    margin-left: 10px;
  }
  .note-name {
    width: 100%;
    padding: 3px 10px;
  }
  .buttons {
    display: flex;
    flex-direction: row;
    .button-icon {
      padding: 3px;
      margin-left: 3px;
      border-radius: 5px;
    }
  }
}
</style>
