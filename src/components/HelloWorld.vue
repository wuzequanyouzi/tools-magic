<template>
  <div id="container"></div>
</template>

<script>
import Vue from 'vue';
export default {
  name: 'HelloWorld',
  props: {
    url: String,
  },
  data() {
    return {};
  },
  mounted() {
    this.importScript();
  },
  methods: {
    async importScript() {
      // 发起 get 请求
      const resCode = await fetch(this.url).then((res) => res.text());
      this.sandboxEval(resCode);
    },
    sandboxEval(code) {
      const fakeWindow = {};
      const proxyWindow = new Proxy(window, {
        // 获取属性
        get(target, key) {
          // https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol/unscopables
          if (key === Symbol.unscopables) return false;

          // 内部可能访问当这几个变量，都直接返回代理对象
          if (['window', 'globalThis'].includes(key)) {
            return proxyWindow;
          } else if (['self'].includes(key)) {
            return window;
          }

          return target[key] || fakeWindow[key];
        },
        // 设置属性
        set(target, key, value) {
          return (fakeWindow[key] = value);
        },
        // 判断属性是否有
        has(target, key) {
          return key in target || key in fakeWindow;
        },
      });
      const tempPproxyWindow = window.proxyWindow;
      window.proxyWindow = proxyWindow;

      // 这是一个自执行函数
      // 并且通过 `call` 调用，因为 code 可能通过 this 访问 window，所以通过 call 改变 this 指向
      const codeBindScope = `
        (function (window) {
          with (window) {
            ${code}
          }
        }).call(window.proxyWindow, window.proxyWindow)
      `;

      // 通过 new Function 的方式执行
      const fn = new Function(codeBindScope);
      fn();

      // const Com = Vue.extend(window.proxyWindow['test-umd'].default);
      new Vue({
        render: (h) =>
          h(window.proxyWindow['test-umd'].default, {
            props: this.$attrs,
            on: this.$listeners,
          }),
      }).$mount('#container');

      tempPproxyWindow && (window.proxyWindow = tempPproxyWindow);
    },
  },
};
</script>

<style scoped lang="scss"></style>
