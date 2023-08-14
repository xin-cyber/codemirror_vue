<template>
  <div id="app">
    <!-- 下面的:code可以用:value或:content代替 -->
    <codemirror
      ref="cm"
      :code="code"
      :options="cmOptions"
      @input="inputChange"
    ></codemirror>
  </div>
</template>

<script>
// 全局引入vue-codemirror
import { codemirror } from "vue-codemirror";
import "codemirror/lib/codemirror.css";
import "codemirror/lib/codemirror.js";
import "codemirror/mode/cypher/cypher.js";

import "codemirror/addon/hint/show-hint.css";
import "codemirror/addon/hint/show-hint.js";

import { keys } from "./KeyWord";
export default {
  name: "App",
  components: { codemirror },
  data() {
    return {
      // MATCH (n) RETURN n LIMIT 100
      code: "",
      cmOptions: {
        mode: "cypher",
        line: true,
        lineNumbers: true,
        tabSize: 4,
        hintOptions: {
          completeSingle: false,
          hint: this.handleShowHint,
        },
      },
    };
  },
  methods: {
    inputChange(content) {
      this.$nextTick(() => {
        // console.log("code:" + this.code);
        // console.log("content:" + content);
      });
    },
    // 实例 ，配置
    handleShowHint(cmInstance, hintOptions) {
      // 光标实例，光标实例里有行数、列数
      let cursor = cmInstance.getCursor();
      // 传入一个行数，从而获取行中的字符串
      let cursorLine = cmInstance.getLine(cursor.line);
      let result = [];
      //token 中的 start和end指的是光标所在字符串在这一行的起始位置和结束位置
      let token = cmInstance.getTokenAt(cursor),
        start,
        end,
        search;
      console.log(token, cursor);

      if (token.end > cursor.ch) {
        token.end = cursor.ch;
        token.string = token.string.slice(0, cursor.ch - token.start);
      }
      // 不包含特殊字符直接赋值
      if (token.string.match(/^[.`"'\w@][\w$#]*$/g)) {
        search = token.string;
        start = token.start;
        end = token.end;
      } else {
        start = end = cursor.ch;
        search = "";
      }
      if (!search.charAt(0) || search.charAt(0) === ".") {
        // start = nameCompletion(cur, token, result, editor);
      } else {
        console.log("输入值：", search);
        result = keys.filter((item) => new RegExp(search, "i").test(item));
      }
      console.log(
        result.map((item) => {
          return {
            text: item,
            displayText: item,
            render: this.hintRender,
          };
        })
      );

      return {
        list: result.map((item) => {
          return {
            text: item,
            displayText: item,
            render: this.hintRender,
          };
        }),
        from: { ch: start, line: cursor.line },
        to: { ch: end, line: cursor.line },
      };
    },
    // 自定义渲染选项
    hintRender(element, self, data) {
      let div = document.createElement("div");
      div.setAttribute("class", "container");

      let divText = document.createElement("div");
      divText.setAttribute("class", "key");
      divText.innerText = data.displayText;

      div.appendChild(divText);
      element.appendChild(div);
    },
  },
  mounted() {
    // 代码提示功能 当用户有输入时，显示提示信息
    this.$refs.cm.codemirror.on("inputRead", (cm) => {
      cm.showHint();
    });
  },
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.container {
  display: inline-block;
  width: 100%;
}
.key {
  display: inline-block;
  font-size: 11px;
  /* color: blue; */
}
</style>
