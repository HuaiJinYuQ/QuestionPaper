<!--
程序名：模板问卷预览页面
功能：预览问卷模板
-->
<template>
  <div class="Design" v-loading="loading" element-loading-text="加载中...">
    <h3>{{ this.title }}</h3>
    <div class="top" v-if="desc != ''">
      {{ desc }}
    </div>
    <el-card
      class="box-card"
      :key="index"
      v-for="(item, index) in detail"
      style="margin: 10px;"
    >
      <div slot="header" class="clearfix">
        <div class="questionTitle">
          <!--显示必填标识-->
          <span style="color: #F56C6C;">
            <span v-if="item.isMust">*</span>
            <span v-else>&nbsp;</span>
          </span>
          <span style="color: black;margin-right: 3px;">{{
            index + 1 + "."
          }}</span>
          {{ item.mainTitle }}
          <el-divider content-position="left">{{ item.subtitle }}</el-divider>
        </div>
      </div>

      <!--单选题展示-->
      <div
        class="text item"
        :key="index"
        v-for="(option, index) in item.options"
      >
        <div v-if="item.questionType == '2'">
          <el-radio
            v-model="item.radioValue"
            :label="index"
            style="margin: 5px;"
            >{{ option }}</el-radio
          >
        </div>
      </div>

      <!--多选题展示-->
      <el-checkbox-group
        v-if="item.questionType == '3'"
        v-model="item.checkboxValue"
      >
        <div
          class="text item"
          :key="index"
          v-for="(option, index) in item.options"
        >
          <el-checkbox :label="index" style="margin: 5px;">{{
            option
          }}</el-checkbox>
        </div>
      </el-checkbox-group>

      <!--填空题展示-->
      <el-input
        v-if="item.questionType == '1'"
        type="textarea"
        resize="none"
        v-model="item.textValue"
      >
      </el-input>
    </el-card>

    <el-button type="primary" style="margin-top: 10px;">保存</el-button>

    <br /><br /><br /><br /><br />
  </div>
</template>
<script>
import qs from "qs";
export default {
  data() {
    return {
      loading: false, //页面加载中
      dialogShow: false,
      dialogTitle: "",
      detail: [],
      sn: "", //问卷SN
      title: "", //问卷标题
      desc: "", //问卷备注
      xsWidows: false, //当前是否为手机打开
      dialogWidth: 50,
      willAddQuestion: {
        //保存问卷问题信息
        paperSN: this.sn, //问卷SN
        questionType: "", //问题类型
        mainTitle: "", //问题主标题
        subtitle: "", //问题副标题
        option: [],
        options: [
          {
            title: ""
          }
        ],
        sortOrder: 1, //问题顺序
        isMust: false //是否必填
      },
      allType: [
        {
          value: "2",
          label: "单选题"
        },
        {
          value: "3",
          label: "多选题"
        },
        {
          value: "1",
          label: "填空题"
        }
        // {
        //   value: "4",
        //   label: "评分题"
        // }
      ]
    };
  },
  mounted() {
    //初始化问卷所有问题

    this.sn = this.$route.params.sn;
    debugger;
    this.$axios.get(`/modular/${this.sn}`).then(data => {
      this.title = data.data.data.queTitle;
      this.desc = data.data.data.queDescribe;
    });
    this.getQuestionList();
  },
  methods: {
    //获取问题列表(问卷内容)
    getQuestionList() {
      this.detail = [];
      this.loading = true;
      this.paperSN = this.sn;
      debugger;
      this.$axios.get(`/qub/${this.paperSN}`).then(data => {
        let datas = data.data.data;
        this.detail = datas;
        if (this.detail == null || this.detail == "") {
          this.loading = false;
          return;
        }
        this.detail.forEach(t => {
          if (this.isSelectQuestionType(t.questionType)) {
            if (t.options != null) {
              if (t.options.size != 0) {
                t.options.shift();
              }
            }
          }
        });
        this.loading = false;
      });
    },

    //判断是否为选择题
    isSelectQuestionType(questionType) {
      if (questionType == 2 || questionType == 3) {
        return true;
      }
      return false;
    }
  }
};
</script>
<style scoped>
.Design {
  padding: 0%;
}
.Design .dialog {
  text-align: left;
}
.Design .questionTitle {
  display: inline-block;
  width: 80%;
  font-size: 16px;
  color: #303133;
}
.Design .addOptionButton {
  display: inline-block;
  margin-left: 80px;
}
.box-card {
  width: 100%;
  text-align: left;
}
.Design .top {
  color: #606266;
  margin-left: 20px;
  padding: 0 10px 10px 10px;
  border-bottom: 3px solid #409eff;
  font-size: 15px;
  line-height: 22px;
  text-align: left;
}
</style>
