<!--
程序名：用户管理
功能：管理网站的所有用户
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
        <el-button plain type="primary" icon="el-icon-plus" @click="addUser()"
          >添加问卷</el-button
        >
      </el-col>
    </el-col>
    <el-table
      :data="modularList"
      :default-sort="{ prop: 'date', order: 'descending' }"
      :height="screenHeight"
      border
    >
      <el-table-column prop="sn" sortable label="编号"> </el-table-column>
      <el-table-column prop="queTitle" label="问卷标题"> </el-table-column>
      <el-table-column prop="useCount" label="使用次数"> </el-table-column>
      <el-table-column prop="queDescribe" label="问卷描述"> </el-table-column>
      <!--<el-table-column label="启用状态">
        <template slot-scope="scope">
          <el-switch
            v-model="scope.row.enabledMark"
            active-text="是"
            inactive-text="否"
            @change="changeEnabledMark(scope.$index, scope.row)"
          ></el-switch>
        </template>
      </el-table-column>-->
      <el-table-column align="right" fixed="right" width="300">
        <template slot="header">
          <el-autocomplete
            class="input-with-select"
            v-model="queryString"
            size="mini"
            :fetch-suggestions="querySearch"
            placeholder="输入标题搜索"
            :trigger-on-focus="true"
            @select="handleSelect"
            clearable>
          </el-autocomplete>
        </template>
        <template slot-scope="scope">
          <el-button size="mini" @click="handleEdit(scope.$index, scope.row)"
            >编辑问卷</el-button>
          <el-button
            size="mini"
            type="warning"
            @click="goProblemData(scope.row)"
            >添加问卷问题</el-button>
          <el-button
            size="mini"
            type="danger"
            @click="handleDelete(scope.$index, scope.row)"
            >删除</el-button>
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
        :total="records"
        style="float:right;"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
      >
      </el-pagination>
    </el-col>

    <!--保存用户界面-->
    <el-dialog :title="fromTitle" :visible.sync="formVisible" width="50%">
      <el-form
        :model="profile"
        label-width="80px"
        :rules="profileRules"
        ref="profile"
      >
        <el-form-item label="问卷标题" prop="queTitle">
          <el-input v-model="profile.queTitle" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="问卷表述" prop="queDescribe">
          <el-input v-model="profile.queDescribe"></el-input>
        </el-form-item>
        <el-form-item label="备注" prop="remarks">
          <el-input v-model="profile.remarks"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click.native="formVisible = false">取消</el-button>
        <el-button
          type="primary"
          @click.native="formSubmit()"
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
      modularList: [], //用户信息
      queryModularList: [], //用户信息 用户搜索框查询使用
      queryString: "", //搜索条件
      screenHeight: window.innerHeight - 220, //当前窗体高度-导航栏高度
      //分页信息
      page: 1, //当前页
      rows: 10, //页面数据数量
      total: 1, //页数
      records: 10, //总数
      profile: {
        sn: "", //问卷标识
        queTitle: "", //问卷标题
        queDescribe: "", //问卷描述
        remarks: "", //备注
      },
      profileRules: {
        //表单用户信息的格式显示
        queTitle: [
          {
            required: true,
            message: "问卷标题必须填写!!",
            trigger: "blur"
          }
        ]
      },
      formVisible: false, //表单的打开和关闭的控制
      formLoading: false, //表单
      fromTitle: "新增问卷" //
    };
  },
  methods: {
    //前往添加问卷问题页面
    goProblemData(row) {
      this.$router.push({ path: `/management/problemData/${row.sn}` });
    },
    //删除用户信息
    handleDelete(index, row) {
      this.$confirm("确认要删除此问卷模板信息吗？", "提示", {}).then(() => {
        this.$axios.delete(`/modular/${row.sn}`).then(res => {
          if (res.data.code == 200) {
            this.$message({
              type: "success",
              message: "删除成功!"
            });
            this.loading = false;
            this.getAllModularList();
            this.defaultActive = 1;
          } else {
            this.$message({
              type: "error",
              message: data.msg
            });
          }
        });
      });
    },
    //搜索
    querySearch(queryString, cb) {
      if (queryString == "" || queryString == null) {
        this.getAllModularList();
      }

      this.getAllModular();
      let modularList = this.queryModularList;
      let modulars = queryString
        ? modularList.filter(this.createFilter(queryString))
        : modularList;
      let modularValue = [];
      modulars.forEach(modular => {
        modularValue.push({ value: modular.queTitle });
      });
      // 调用 callback 返回建议列表的数据
      cb(modularValue);
    },
    createFilter(queryString) {
      return modular => {
        return (
          modular.queTitle.toLowerCase().indexOf(queryString.toLowerCase()) !== -1
        );
      };
    },
    handleSelect(item) {
      console.log(item);
      this.queryString = item.value;
      this.getAllModularList();
    },
    //获取用户列表
    getAllModularList() {
      this.$axios
        .get("/modular/queryGridJson", {
          params: {
            queryJson: this.queryString,
            page: this.page, //当前页
            rows: this.rows, //页面数据数量
            total: this.total, //页数
            records: this.records, //总数
            code: ""
          }
        })
        .then(res => {
          let data  = res.data.data;
          if (data  !== null) {
            this.modularList = data.rows;
            this.page = data.page;
            this.rows = data.pagesize;
            this.total = data.total;
            this.records = data.records;
          }
        });
    },
    //获取所有的用户信息，不包含查询和分页
    getAllModular() {
      this.$axios
        .get("/modular/queryGridJson", {
          params: {
            rows: 99999 //页面数据数量
          }
        })
        .then(res => {
          this.queryModularList = res.data.data.rows;
        });
    },
    //翻页
    handleSizeChange(size) {
      this.rows = size;
      this.getAllModularList();
    },
    handleCurrentChange(val) {
      this.page = val;
      this.getAllModularList();
    },
    //保存问卷模板信息
    formSubmit() {
      this.$refs.profile.validate(valid => {
        console.log(valid);
        if (valid) {
          this.$confirm("确认提交吗？", "提示", {}).then(() => {
            this.formLoading = true;
            debugger;
            this.$axios
              .post("/modular/savePaperTemplate", this.profile)
              .then(res => {
                debugger;
                this.formLoading = false;
                if (res.data.code == 200) {
                  this.$message({
                    message: "提交成功",
                    type: "success"
                  });
                  this.$refs["profile"].resetFields();
                  this.getAllModularList();
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
    //打开添加用户表单
    addUser() {
      this.fromTitle = "新增问卷";
      //初始化表单
      this.profile = {
        remarks: "", //备注
        queTitle: "", //问卷标题
        queDescribe: "", //问卷描述
        sn: "" //用戶SN
      };
      this.formVisible = true;
    },
    //打开编辑用户表单
    handleEdit(index, row) {
      this.fromTitle = "编辑问卷信息";
      //初始化表单
      this.profile = {
        remarks: row.remarks, //备注
        queTitle: row.queTitle, //问卷标题
        queDescribe: row.queDescribe, //问卷描述
        sn: row.sn //用戶SN
      };
      this.formVisible = true;
    },
    //更改账户启用状态
    changeEnabledMark(index, row) {
      this.$confirm("是否确认更改当前用户的启用状态？", "提示", {
        distinguishCancelAndClose: true,
        confirmButtonText: "确定",
        cancelButtonText: "取消"
      })
        .then(() => {
          if (row.enabledMark) {
            this.$axios.post(`/user/enabledUser/${row.sn}`).then(res => {
              let resCode = res.data.code;
              if (resCode == 200) {
                this.$message({
                  type: "success",
                  message: "启用成功"
                });
              } else if (resCode == 400) {
                this.$message.error("启用失败");
              }
            });
          } else {
            this.$axios.post(`/user/disabledUser/${row.sn}`).then(res => {
              let resCode = res.data.code;
              if (resCode == 200) {
                this.$message({
                  type: "success",
                  message: "停用成功"
                });
              } else if (resCode == 400) {
                this.$message.error("停用失败");
              }
            });
          }
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消更改"
          });
          this.getAllModularList();
        });
    }
  },
  mounted() {
    //监听当前窗口高度
    window.onresize = () => {
      return (() => {
        this.screenHeight = window.innerHeight - 220;
      })();
    };
    this.getAllModularList();
  }
};
</script>
<style></style>
