<template>
  <div class="main-page">
    <div class="left-menu" @click.self="onEditNoteEnd()">
      <!-- ノートリスト -->
      <draggable v-bind:list="noteList" group="notes">
        <!-- v-bind = コンポーネントに値を受け渡しを行うディレクティブ -->
        <!-- @deleteイベントが発覚したタイミングでonDeleteNoteメソッドを呼び出す -->
        <NoteItem
          v-for="note in noteList"
          v-bind:note="note"
          v-bind:key="note.id"
          v-bind:layer="1"
          @delete="onDeleteNote"
          @select="onSelectNote"
          @editStart="onEditNoteStart"
          @editEnd="onEditNoteEnd"
          @addChild="onAddChildNote"
          @addNoteAfter="onAddNoteAfter"
        />
      </draggable>
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
    <div class="right-view" @click.self="onEditNoteEnd()">
      <template v-if="selectedNote == null">
        <div class="no-selected-note">ノートを選択してください</div>
      </template>
      <template v-else>
        <div class="path">
          <small>{{ selectedPath }}</small>
        </div>
        <div class="note-content">
          <h3 class="note-title">{{ selectedNote.name }}</h3>
          <draggable v-bind:list="selectedNote.widgetList" group="widgets">
            <WidgetItem
              v-for="widget in selectedNote.widgetList"
              v-bind:widget="widget"
              v-bind:layer="1"
              v-bind:key="widget.id"
              @delete="onDeleteWidget"
              @addChild="onAddChildWidget"
              @addWidgetAfter="onAddWidgetAfter"
            />
          </draggable>
          <button class="transparent" @click="onClickButtonAddWidget">
            <i class="fas fa-plus-square"></i>ウィジェットを追加
          </button>
        </div>
      </template>
    </div>
  </div>
</template>

<script>
import NoteItem from "@/components/parts/NoteItem.vue";
import WidgetItem from "@/components/parts/WidgetItem.vue";
import draggable from "vuedraggable";

export default {
  data() {
    return {
      // 1. 変数の定義
      noteList: [],
      selectedNote: null,
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
        selected: false,
        children: [],
        layer: layer,
        widgetList: [],
      };
      // ノートの作成と同時にウィジェットを1つ作成
      this.onAddWidgetCommon(note.widgetList);
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
    onSelectNote: function (targetNote) {
      // 再帰的にノートの選択状態を更新
      const updateSelectStatus = function (targetNote, noteList) {
        for (let note of noteList) {
          // 選択されたノートと一致すればselected = true。それ以外はselected = falseに更新
          note.selected = note.id === targetNote.id;
          updateSelectStatus(targetNote, note.children);
        }
      };
      updateSelectStatus(targetNote, this.noteList);

      // 選択中ノート情報を更新
      this.selectedNote = targetNote;
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
    // ノート内にウィジェットを追加
    onAddWidgetCommon: function (targetList, layer, index) {
      layer = layer || 1;
      const widget = {
        id: new Date().getTime().toString(16),
        /* 
        メインページ側でウィジェットを追加する際に、
        階層数が1の場合には見出しとして作成しそれ以外では本文として作成する
        */
        type: layer === 1 ? "heading" : "body",
        text: "",
        mouseover: false,
        children: [],
        layer: layer,
      };
      if (index == null) {
        targetList.push(widget);
      } else {
        targetList.splice(index + 1, 0, widget);
      }
    },
    onClickButtonAddWidget: function () {
      this.onAddWidgetCommon(this.selectedNote.widgetList);
    },
    onAddChildWidget: function (widget) {
      this.onAddWidgetCommon(widget.children, widget.layer + 1);
    },
    onAddWidgetAfter: function (parentWidget, note) {
      const targetList =
        parentWidget == null
          ? this.selectedNote.widgetList
          : parentWidget.children;
      const layer = parentWidget == null ? null : parentWidget.layer + 1;
      const index = targetList.indexOf(note);
      this.onAddWidgetCommon(targetList, layer, index);
    },
    onDeleteWidget: function (parentWidget, widget) {
      const targetList =
        parentWidget == null
          ? this.selectedNote.widgetList
          : parentWidget.children;
      const index = targetList.indexOf(widget);
      targetList.splice(index, 1);
    },
  },
  /* computedオプション
  関数で計算した結果を返すということで、methodsオプションとよく類似して比較されるのですが以下のような違いがあります。
  methods
  1. 変数の更新／取得が可能
  2. 呼び出しのたびに処理が行われる
  3.引数を渡すことが可能
  4. template内では関数名()の形で呼び出し
  
  computed
  1. 変数の取得のみ可能（※ 別途対応で更新も可能）
  2. 計算に利用している変数が更新されない限り、再計算は行われない
  3. 引数を渡すことができない
  4. template内では関数名の形で呼び出し

  そのため、引数を必要とせず、特定のルールに則り値を返すような関数に関してはcomputedを使い、
  それ以外はmethodsを使うような場合分けが良いかと思われます。
  */
  // ノートまでのパスの計算
  computed: {
    selectedPath: function () {
      const searchSelectedPath = function (noteList, path) {
        for (let note of noteList) {
          const currentPath =
            path == null ? note.name : `${path} / ${note.name}`;
          if (note.selected) return currentPath;
          const selectedPath = searchSelectedPath(note.children, currentPath);
          if (selectedPath.length > 0) return selectedPath;
        }
        return "";
      };
      return searchSelectedPath(this.noteList);
    },
  },
  components: {
    NoteItem,
    draggable,
    WidgetItem,
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
    .no-selected-note {
      text-align: center;
      font-size: 25px;
      margin: 20px;
    }
    .path {
      text-align: left;
      margin-bottom: 50px;
    }
    .note-content {
      max-width: 900px;
      margin: 0 auto;
      text-align: left;
      .note-title {
        margin-bottom: 25px;
      }
    }
  }
}
</style>
