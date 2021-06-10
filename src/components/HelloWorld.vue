<template>
  <div>
    <editor v-model="myValue" :init="init" :disabled="disabled"></editor>
  </div>
</template>

<script>
import axios from 'axios';
import tinymce from "tinymce";
import Editor from "@tinymce/tinymce-vue";
import "tinymce/themes/silver";
import "tinymce/plugins/paste";
import "tinymce/plugins/image";
import "tinymce/plugins/link";
import "tinymce/plugins/code";
import "tinymce/plugins/table";
import "tinymce/plugins/lists";
import "tinymce/plugins/wordcount";
import "tinymce/icons/default";
import "tinymce/plugins/indent2em";
export default {
  components: {
    Editor
  },
  props: {
    value: {
      type: String,
      default: ""
    },
    disabled: {
      type: Boolean,
      default: false
    },
    plugins: {
      type: [String, Array],
      default:
        "link lists image code table wordcount indent2em"
    },
    toolbar: {
      type: [String, Array],
      default:
        "bold italic underline strikethrough | fontsizeselect fontselect | forecolor backcolor | alignleft aligncenter alignright alignjustify | bullist numlist indent2em | outdent indent blockquote | undo redo | link unlink code | removeformat"
    }
  },
  data() {
    return {
      public: process.env.BASE_URL,
      init: {
        language_url: process.env.BASE_URL + "tinymec/langs/zh_CN.js", //如果语言包不存在，指定一个语言包路径
        language: "zh_CN", //语言
        skin_url: process.env.BASE_URL + "tinymec/skins/ui/oxide", //如果主题不存在，指定一个主题路径
        // content_css: "/static/plugins/tinymce/mycontent.css",
        height: "700px",
        width: this.calcWidth(),
        plugins: this.plugins, //插件
        toolbar: this.toolbar, //工具栏
        branding: false, //技术支持(Powered by Tiny || 由Tiny驱动)
        menubar: true, //菜单栏
        theme: "silver", //主题
        zIndex: 1101,
        images_upload_handler: (blobInfo, success, failure) => {
         this.upImg(blobInfo, success, failure)
        }
      },
      myValue: this.value
    };
  },
  mounted() {
    console.log(this.public);
    tinymce.init({});
  },
  methods: {
    calcWidth() {
      return document.body.clientWidth / 2 + "px";
    },
    upImg(blobInfo, success, failure) {
      var formData = new FormData();
      formData.append("img", blobInfo.blob());
      formData.append("type", 2);
      axios({
        method: "post",
        url: "http://192.168.12.138:8800/admin/upload/upload",
        data: formData,
        headers: {token: "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoie1wiaWRcIjoxLFwidXNlcm5hbWVcIjpcImFkbWluXCIsXCJuaWNrbmFtZVwiOlwiY2RzZndlXCIsXCJyZWFsbmFtZVwiOlwiXFx1OTE5MlxcdTkxOTJkZGRhXCIsXCJhdmF0YXJcIjpcIlxcXC8yMDIxMDZcXFwvMDNcXFwvOWRkYmM2Zjg4MjQwNjc1ZTE0YmQyM2YzYzU2ZTViY2QucG5nXCIsXCJnZW5kZXJcIjoyLFwiZ3JvdXBfaWRcIjoxLFwic3RhdHVzXCI6MSxcImxhc3RfbG9naW5fdGltZVwiOlwiMjAyMS0wNi0wMyAxNTozMDoxNFwiLFwibGFzdF9sb2dpbl9pcFwiOlwiMTcxLjIxNC4xODkuOTFcIn0ifQ.47GULp_I-5OUx_we-IGNloX0T7IZXDuasJXFWL5ZUo0"}
      }).then(res => {
        console.log(res.data);
        success(res.data.data.rel_path);
        console.log(failure);
      })
    }
  },
  watch: {
    value(newValue) {
      this.myValue = newValue;
      console.log(newValue);
    },
    myValue(newValue) {
      this.$emit("input", newValue);
      console.log(newValue);
    }
  }
};
</script>