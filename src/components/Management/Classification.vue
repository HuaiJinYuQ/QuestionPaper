<!--
程序名：问题分类管理
功能：管理网站的所有问题分类
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
        <el-button plain type="primary" icon="el-icon-plus" @click="addClassification('添加问题分类')"
          >添加问题分类</el-button
        >
      </el-col>
    </el-col>
    <el-table
      :data="classificationList"
      :default-sort="{ prop: 'date', order: 'descending' }"
      :height="screenHeight"
      border
    >
      <el-table-column prop="sn" sortable label="问题分类标识" width="250">
      </el-table-column>
      <el-table-column prop="categoryName" label="问题分类名称" width="200">
      </el-table-column>
      <el-table-column prop="remarks" label="备注">
      </el-table-column>
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
          <el-button size="mini" @click="updateClassification('编辑问题分类', scope.row)"
            >编辑</el-button
          >
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
    <!--保存问题分类界面-->
    <el-dialog :title="fromTitle" :visible.sync="formVisible">
      <el-form
        :model="profile"
        label-width="80px"
        :rules="profileRules"
        ref="profile"
      >
        <el-form-item label="分类名称" prop="categoryName">
          <el-input v-model="profile.categoryName" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="备注" prop="remarks">
          <el-input v-model="profile.remarks"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click.native="formVisible = false">取消</el-button>
        <el-button
          type="primary"
          @click.native="formSubmit('profile')"
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
      classificationList: [], //问题分类信息
      queryJson: "", //搜索条件
      screenHeight: window.innerHeight - 220, //当前窗体高度-导航栏高度
      currentPage: 1,
      pagesize: 10,
      total: 0,
      profile: {
        sn: "", //分类标识
        categoryName: "", //分类名称
        enabledMark: "", //是否启用
        remarks: "" //备注
      },
      profileRules: {
        //表单用户信息的格式显示
        categoryName: [
          {
            required: true,
            message: "分类名称必须填写!!",
            trigger: "blur"
          }
        ]
      },
      formVisible: false, //表单的打开和关闭的控制
      formLoading: false, //表单
      fromTitle: "" //
    };
  },
  components: {},
  methods: {
    //删除问题分类信息
    handleDelete(index, row) {
      this.$confirm("确认要删除此问题分类信息吗？", "提示", {}).then(() => {
        this.$axios.delete(`/type/${row.sn}`).then(res => {
          if (res.data.code == 200) {
            this.$message({
              type: "success",
              message: "删除成功!"
            });
            this.loading = false;
            this.getAllClassificationList();
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
      this.getAllClassificationList();
    },
    //获取问题分类列表
    getAllClassificationList() {
      this.$axios.get("/type/selectAll").then(res => {
        this.classificationList = res.data.data;
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
    //保存问题分类信息
    formSubmit() {
      this.$refs.profile.validate(valid => {
        console.log(valid);
        if (valid) {
          this.$confirm("确认提交吗？", "提示", {}).then(() => {
            this.formLoading = true;
            this.$axios.post("/type/submitForm",this.profile).then(res => {
                debugger;
              this.formLoading = false;
              if (res.data.code == 200) {
                this.$message({
                  message: "提交成功",
                  type: "success"
                });
                this.$refs["profile"].resetFields();
                this.getAllClassificationList();
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
    //打开添加问题分类表单
    addClassification(fTitle) {
      this.fromTitle = fTitle;
      this.profile.categoryName=null;
      this.profile.remarks=null;
      this.profile.enabledMark=true;
      this.profile.sn=null;
      this.formVisible = true;
    },
    //打开编辑问题分类表单
    updateClassification(fTitle,row) {
      this.fromTitle = fTitle;
      this.profile.categoryName=row.categoryName;
      this.profile.remarks=row.remarks;
      this.profile.sn=row.sn;
      this.profile.enabledMark=true;
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
    this.getAllClassificationList();
  }
};
</script>
<style></style>
