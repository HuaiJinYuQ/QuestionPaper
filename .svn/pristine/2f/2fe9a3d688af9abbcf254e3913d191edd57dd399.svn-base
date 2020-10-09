<!--
程序名：修改密码
功能：用户用户修改用户名密码
-->
<template>
  <el-form
    :rules="rules"
    :model="passwordFrom"
    style="margin: 20px;"
    ref="passwordFrom"
  >
    <el-form-item label="原始密码" prop="oldPassword">
      <el-input
        v-model="passwordFrom.oldPassword"
        show-password
        @keyup.enter.native="submitForm('passwordFrom')"
      ></el-input>
    </el-form-item>
    <el-form-item label="新密码" prop="newPassword">
      <el-input
        v-model="passwordFrom.newPassword"
        show-password
        @keyup.enter.native="submitForm('passwordFrom')"
      ></el-input>
    </el-form-item>
    <el-form-item label="确认密码" prop="againNewPassword">
      <el-input
        v-model="passwordFrom.againNewPassword"
        show-password
        @keyup.enter.native="submitForm('passwordFrom')"
      ></el-input>
    </el-form-item>
    <el-form-item>
      <el-button type="primary" @click="submitForm('passwordFrom')"
        >提交</el-button
      >
    </el-form-item>
  </el-form>
</template>

<script>
export default {
  data() {
    let checkAgainNewPassword = (rule, value, callback) => {
      if (!value) {
        return callback(new Error("不能为空"));
      } else if (value != this.passwordFrom.newPassword) {
        return callback(new Error("密码不一致,请重新输入!!"));
      } else {
        callback();
      }
    };
    return {
      passwordFrom: {
        oldPassword: "", //旧密码
        newPassword: "", //新密码
        againNewPassword: "" //重复一次新密码
      },
      rules: {
        oldPassword: [
          { required: true, message: "请输入原始密码", trigger: "blur" }
        ],
        newPassword: [
          { required: true, message: "密码不能为空", trigger: "blur" }
        ],
        againNewPassword: [
          { required: true, validator: checkAgainNewPassword, trigger: "blur" }
        ]
      }
    };
  },
  methods: {
    //提交
    submitForm(formName) {
      this.$refs[formName].validate(valid => {
        if (valid) {
          let password = this.$md5(this.passwordFrom.againNewPassword);
          let oldPassword = this.$md5(this.passwordFrom.oldPassword);
          this.$axios
            .post("/user/password", {
              oldPassword: oldPassword,
              newPassword: password
            })
            .then(res => {
              let alertPasswordRes = res.data;
              if (alertPasswordRes.code == 200) {
                this.$message({
                  type: "success",
                  message: "密码修改成功，请重新登录",
                  showClose: true
                });
                sessionStorage.clear(); //清空session
                this.$emit("state"); // 调用state方法
                this.$router.push({ path: "/login" });
              } else {
                if (alertPasswordRes.code == 404) {
                  this.$message({
                    type: "error",
                    message: "原始密码错误",
                    showClose: true
                  });
                } else if (alertPasswordRes.code == 404) {
                  this.$message({
                    type: "error",
                    message: "修改失败,请重试",
                    showClose: true
                  });
                }
              }
            });
        } else {
          return false;
        }
      });
    }
  }
};
</script>

<style></style>
