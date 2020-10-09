<!--
程序名：问题管理
功能：管理网站的问题信息
-->
<template>
  <section>
    <!--工具条-->
    <el-col
      :span="24"
      class="el-table_headtoolbar"
      style="padding-bottom: 15px;"
    >
      <el-col :span="4" :offset="20">
        <el-button plain type="primary" icon="el-icon-plus" @click="addUser"
          >添加问题</el-button
        >
      </el-col>
    </el-col>
    <el-table
      :data="profile.questionList"
      :default-sort="{ prop: 'date', order: 'descending' }"
      :height="screenHeight"
      border
    >
      <el-table-column prop="sn" sortable label="问题编号" width="200">
      </el-table-column>
      <el-table-column prop="mainTitle" label="主标题">
      </el-table-column>
      <el-table-column prop="subtitle" label="副标题" width="200">
      </el-table-column>
      <el-table-column prop="questionType" label="问题类型" width="100">
        <span v-if="profile.questionType==1">简答题</span>
        <span v-else-if="profile.questionType==2">单选题</span>
        <span v-else-if="profile.questionType==3">多选题</span>
      </el-table-column>
      <el-table-column prop="answer" label="标准答案"> </el-table-column>
      <el-table-column prop="remarks" label="备注"> </el-table-column>
      <el-table-column
        prop="enabledMark"
        label="启用状态"
        :formatter="enabledMarkFormat"
        width="100"
      >
      </el-table-column>
      <el-table-column align="right" fixed="right" width="200">
        <template slot="header" slot-scope="scope">
          <el-input
            v-model="queryJson"
            size="mini"
            placeholder="输入关键字搜索"
            :input="searchRole(scope)"
          />
        </template>
        <template slot-scope="scope">
          <el-button size="mini" @click="resetPassword(scope.$index, scope.row)"
            >编辑</el-button
          >
          <!--<el-button size="mini" @click="handleEdit(scope.$index, scope.row)"
            >编辑</el-button
          >-->
          <el-button
            size="mini"
            type="danger"
            @click="handleDelete(scope.$index, scope.row)"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>
    <!--翻页栏-->
    <el-col :span="24" class="el-table_footertoolbar">
      <el-pagination
        :hide-on-single-page="false"
        background
        layout="total, sizes, prev, pager, next"
        :page-sizes="[10, 20, 50, 100]"
        :total="total"
        style="float:right;"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
      >
      </el-pagination>
    </el-col>
    <!--保存问题界面-->
    <el-dialog :title="fromTitle" :visible.sync="formVisible">
      <el-form
        :model="profile"
        label-width="80px"
        :rules="profileRules"
        ref="profile"
      >
        <el-form-item label="主标题" prop="mainTitle">
          <el-input v-model="profile.account" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="副标题" prop="subtitle">
          <el-input v-model="profile.fullName"></el-input>
        </el-form-item>
        <el-form-item label="问题类型" prop="questionType">
          <el-radio-group v-model="radio">
            <el-radio :label="1">简答题</el-radio>
            <el-radio :label="2">单选题</el-radio>
            <el-radio :label="3">多选题</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="问题分类:" prop="username">
          <el-select v-model="questionClass.categoryName" placeholder="请选择" disabled  @change="selectGetObjectSN($event)" style="width:700px">
            <el-option
              v-for="item in users"
              :key="item.sn"
              :label="item.categoryName"
              :value="item.sn">
            </el-option>
          </el-select>
        </el-form-item>
        
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click.native="formVisible = false">取消</el-button>
        <el-button
          type="primary"
          @click.native="formSubmit"
          :loading="formLoading"
          >提交</el-button
        >
      </div>
    </el-dialog>
  </section>
</template>

<script>
export default {
  data() {
    return {
      userList: [], //问题信息
      queryJson: "", //搜索条件
      screenHeight: window.innerHeight - 220, //当前窗体高度-导航栏高度
      currentPage: 1,
      pagesize: 10,
      total: 0,
      profile: {
        questionList: [],
        sn: "", //问题标识
        mainTitle: "", //主标题
        subtitle: "", //副标题
        questionType: "", //问题类型
        answer:"",//答案
        remarks:""//备注
      },
      questionClass:{
        sn:"",
        categoryName:""
      },
      profileRules: {
        //表单用户信息的格式显示
        mainTitle: [
          {
            required: true,
            message: "主标题必须填写!!",
            trigger: "blur"
          }
        ]
      },
      formVisible: false, //表单的打开和关闭的控制
      formLoading: false, //表单
      fromTitle: "新增用户" //
    };
  },
  components: {},
  methods: {
    //删除问题信息
    handleDelete(index, row) {
      this.$confirm("确认要删除此用户信息吗？", "提示", {}).then(() => {
        this.$axios.delete(`/qub/${row.sn}`).then(res => {
          if (res.data.code == 200) {
            this.$message({
              type: "success",
              message: "删除成功!"
            });
            this.loading = false;
            this.getAllQuestionList();
            this.defaultActive = 1;
          } else {
            this.$message({
              type: "error",
              message: data.msg
            });
          }
        });
      })
    },
    //搜索
    //当输入框内容发生改变时或者失去输入框焦点时触发
    searchRole(scope) {
      console.log(this.searchStr + "搜索");
      this.getAllQuestionList();
    },
    //获取问题列表
    getAllQuestionList() {
      this.$axios.get("/qub/queryGridJson").then(res => {
        this.profile.questionList = res.data.data.rows;
      });
    },
    //数据格式化
    enabledMarkFormat(row, column) {
      if (row.enabledMark === true) {
        return "启用";
      } else {
        return "未启用";
      }
    },
    //初始页page、初始每页数据数pagesize和数据data
    handleSizeChange(size) {
      console.log(size);
      this.pagesize = size;
    },
    handleCurrentChange(val) {
      this.currentPage = val;
    },
    //保存问题信息
    formSubmit() {
      this.$refs.profile.validate(valid => {
        console.log(valid);
        if (valid) {
          this.$confirm("确认提交吗？", "提示", {}).then(() => {
            this.formLoading = true;
            this.$axios.post("/user/saveUserInfo", this.profile).then(res => {
              this.formLoading = false;
              if (data.data.code == 200) {
                this.$message({
                  message: "提交成功",
                  type: "success"
                });
                this.$refs["profile"].resetFields();
                this.getAllQuestionList();
              } else {
                this.$message({
                  type: "error",
                  message: "创建失败!"
                });
              }
              this.formVisible = false;
            });
          });
        }
      });
    },
    //打开表单
    addUser() {
      this.fromTitle = "新增用户";
      this.formVisible = true;
    }
  },
  addUsers(){

  },
  mounted() {
    //监听当前窗口高度
    window.onresize = () => {
      return (() => {
        this.screenHeight = window.innerHeight - 220;
      })();
    };
    this.getAllQuestionList();
  }
};
</script>
<style></style>
