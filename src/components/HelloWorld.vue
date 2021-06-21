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
import 'tinymce/plugins/media';
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
        "link lists image code table wordcount indent2em media"
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
      uploaded: false, //是否上传完成
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
        },
        file_picker_types: "media",
        file_picker_callback: (callback, value, meta) => {
          if(meta.filetype == 'media'){  
            let input = document.createElement('input');//创建一个隐藏的input
            input.setAttribute('type', 'file');
            let that = this;
            input.onchange = function(){
              let file = this.files[0];//选取第一个文件
              that.upMedia(that.qiniuToken, file, 'video'); // 上传视频拿到url
              if(that.uploaded){
                callback(that.resVideo, { title: file.name }) //将url显示在弹框输入框中
              }else{
                setTimeout(()=>{
                  callback(that.resVideo, { title: file.name })
                },2000)
              }
            }
            //触发点击
            input.click();
          }
        },
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
        url: "/admin/upload/upload",
        data: formData,
      }).then(res => {
        console.log(res.data);
        success(res.data.data.rel_path);
        console.log(failure);
      })
    },
    upMedia(callback, value, meta) {
      console.log(callback, value, meta);
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