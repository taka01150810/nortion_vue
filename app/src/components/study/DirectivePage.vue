<template>
  <div class="directive-page">
    <h2>学習2：ディレクティブ</h2>

    <!-- 従来のJavaScript -->
    <h3><span class="badge badge-secondary">JavaScript</span></h3>
    <input id="check" type="checkbox" checked="checked" />
    <span id="text">チェック時だけ見えるよ</span>
    <br />
    <br />

    <!-- Vue.js -->
    <h3><span class="badge badge-secondary">Vue.js</span></h3>
    <input type="checkbox" v-model="checked" />
    <span v-show="checked">チェック時だけ見えるよ</span>
  </div>
</template>

<script>
// 従来のJavaScript
window.onload = function () {
  const checkbox = document.getElementById("check");
  checkbox.onchange = function (event) {
    const checked = event.target.checked;
    const text = document.getElementById("text");
    if (checked) {
      text.style.visibility = "visible";
    } else {
      text.style.visibility = "hidden";
    }
  };
};

export default {
  data() {
    return {
      checked: true,
    };
  },
};

/* v-ifとv-showの違い
Vue.jsのディレクティブとしてv-showと類似したv-ifというものがあり、
どちらも「指定した条件がfalseの場合は要素が見えなくなる」という挙動をします。

一見同じように見えるのですが、以下のような違いがあります。
v-if：条件を満たさないときに「HTML要素自体が存在しなかったこと」になる
v-show：条件を満たさないときに「HTML要素が非表示」になる

表示コストの側面から、
切り替えの頻度が高い場合はv-show、
切り替えの頻度が低い場合にはv-if
を使用するのが良い

*/
</script>

<style scoped>
.directive-page {
  padding: 20px;
}
</style>
