<!--
程序名：审核问卷
功能：审核用户想要发布的问卷
-->
<template>
  <section>
    <!--工具条-->
    <el-col
      :span="24"
      class="el-table_headtoolbar"
      style="padding-bottom: -5px;"
    >
      <el-tabs @tab-click="handleClick" value="all">
        <el-tab-pane label="全部" name="all"></el-tab-pane>
        <el-tab-pane label="已审核" name="1"></el-tab-pane>
        <el-tab-pane label="未审核" name="0"></el-tab-pane>
      </el-tabs>
    </el-col>
    <!-- 工具条结束 -->
    <!-- 表格内容开始 -->
    <el-table
      v-loading="tableLoading"
      :data="paperData"
      :default-sort="{ prop: 'date', order: 'descending' }"
      :height="screenHeight"
      border
    >
      <el-table-column prop="sn" sortable label="问卷编号"> </el-table-column>
      <el-table-column prop="queTitle" label="标题"> </el-table-column>
      <el-table-column prop="createBy" label="创建人"> </el-table-column>
      <el-table-column label="审核状态">
        <template slot-scope="scope">
          <el-switch
            v-model="scope.row.auditStatus"
            active-text="已审核"
            inactive-text="未审核"
            @change="auditPaper(scope.$index, scope.row)"
          ></el-switch>
        </template>
      </el-table-column>
      <el-table-column align="right" fixed="right" width="300">
        <template slot="header">
          <el-autocomplete
            class="input-with-select"
            v-model="paperSearch"
            size="mini"
            :fetch-suggestions="querySearch"
            placeholder="请输入问卷标题"
            :trigger-on-focus="true"
            @select="handleSelect"
            clearable
          >
          </el-autocomplete>
        </template>
        <template slot-scope="scope">
          <el-button size="mini" @click="display(scope.$index, scope.row)"
            >预览</el-button
          >
          <el-button
            size="mini"
            type="warning"
            @click="auditPaper(scope.$index, scope.row)"
            >{{ scope.row.auditStatus ? "取消审核" : "审核" }}</el-button
          >
        </template>
      </el-table-column>
    </el-table>
    <!-- 表格内容结束 -->
    <!--翻页栏开始-->
    <el-col :span="24" class="el-table_footertoolbar">
      <el-pagination
        :hide-on-single-page="false"
        background
        layout="total, sizes, prev, pager, next"
        :page-sizes="[10, 20, 50, 100]"
        :total="paperPage.records"
        style="float:right;"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
      >
      </el-pagination>
    </el-col>
    <!-- 翻页栏结束 -->
  </section>
</template>

<script>
export default {
  data() {
    return {
      paperData: [], //问卷列表
      allPaperData: [], //所有问卷列表，用于搜索使用
      paperSearch: "", //查询问卷关键字
      paperAuditStatus: "", //问卷审核状态
      screenHeight: window.innerHeight - 220, //当前窗体高度-导航栏高度
      paperPage: {
        //问卷分页数据
        page: 1, //当前页
        rows: 10, //页面数据数量
        total: 1, //页数
        records: 10 //总数
      },
      tableLoading: true //加载状态
    };
  },
  methods: {
    //获取所有问卷信息，作为查询时候使用
    getAllPaperData() {
      this.$axios.get("/papers").then(res => {
        this.allPaperData = res.data.data;
      });
    },
    //获取问卷分页信息
    getPagePapers() {
      this.tableLoading = true;
      this.$axios
        .get("/papers", {
          params: {
            paperSearch: this.paperSearch,
            page: this.paperPage.page, //当前页
            rows: this.paperPage.rows, //页面数据数量
            total: this.paperPage.total, //页数
            records: this.paperPage.records, //总数
            auditState: this.paperAuditStatus //审核状态
          }
        })
        .then(res => {
          let resData = res.data.data;
          this.paperData = resData.rows;
          this.paperPage = {
            page: resData.page, //当前页
            rows: resData.pagesize, //页面数据数量
            total: resData.total, //页数
            records: resData.records //总数
          };
        });
      this.tableLoading = false;
    },
    //翻页
    handleSizeChange(size) {
      this.paperPage.rows = size;
      this.getPagePapers();
    },
    handleCurrentChange(val) {
      this.paperPage.page = val;
      this.getPagePapers();
    },
    //搜索
    querySearch(queryString, cb) {
      if (queryString == "" || queryString == null) {
        this.getPagePapers();
      }
      let paperList = this.allPaperData;
      let papers = queryString
        ? paperList.filter(this.createFilter(queryString))
        : paperList;
      let paperSearchData = [];
      papers.forEach(paper => {
        paperSearchData.push({ value: paper.queTitle });
      });
      // 调用 callback 返回建议列表的数据
      cb(paperSearchData);
    },
    createFilter(queryString) {
      return paper => {
        return (
          paper.queTitle.toLowerCase().indexOf(queryString.toLowerCase()) !== -1
        );
      };
    },
    handleSelect(item) {
      console.log(item.value);
      this.paperSearch = item.value;
      this.getPagePapers();
    },
    //审核问卷
    auditPaper(index, row) {
      this.$axios.put(`papers/audit/${row.sn}`).then(res => {
        if (res.data.code == 200) {
          this.$message({
            type: "success",
            message: "操作成功"
          });
        } else if (res.data.code == 400) {
          this.$message({
            type: "error",
            message: "操作失败"
          });
        }
        this.getPagePapers();
      });
    },
    //预览问卷
    display(index, row) {
      let url = `${window.location.origin}/dom/#/display/${row.sn}`; //问卷链接
      window.open(url);
    },
    //选择问卷审核状态
    handleClick(tab, event) {
      switch (tab.name) {
        case "all":
          this.paperAuditStatus = "";
          break;
        case "0":
          this.paperAuditStatus = 0;
          break;
        case "1":
          this.paperAuditStatus = 1;
          break;
        default:
          this.paperAuditStatus = "";
          break;
      }
      this.getPagePapers();
    }
  },
  mounted() {
    window.onresize = () => {
      return (() => {
        this.screenHeight = window.innerHeight - 220;
      })();
    };
    this.getPagePapers();
    this.getAllPaperData();
  }
};
</script>

<style></style>
