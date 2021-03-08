<template>
  <div>
    <el-form
      :rules="rules"
      ref="loginForm"
      v-loading="loading"
      element-loading-text="正在登录..."
      element-loading-spinner="el-icon-loading"
      element-loading-background="rgba(0, 0, 0, 0.8)"
      :model="loginForm"
      class="loginContainer"
    >
      <h3 class="loginTitle">系统登录</h3>
      <el-form-item prop="username">
        <el-input
          size="normal"
          type="text"
          v-model="loginForm.username"
          auto-complete="off"
          placeholder="请输入用户名"
        />
      </el-form-item>
      <el-form-item prop="password">
        <el-input
          size="normal"
          type="password"
          v-model="loginForm.password"
          auto-complete="off"
          placeholder="请输入密码"
        />
      </el-form-item>
      <el-form-item prop="code" style="margin-bottom: 5px">
        <el-input
          size="normal"
          type="text"
          v-model="loginForm.code"
          auto-complete="off"
          placeholder="点击图片更换验证码"
          @keydown.enter.native="submitLogin"
          style="width: 250px"
        />
        <img
          :src="vcUrl"
          @click="updateVerifyCode"
          alt=""
          style="cursor: pointer"
        />
      </el-form-item>
      <!-- <el-checkbox size="normal" class="loginRemember" v-model="checked"></el-checkbox> -->
      <div style="text-align: right; margin-bottom: 5px">
        <el-checkbox v-model="remeberme">自动登录</el-checkbox>
        <el-button
          style="size: mini; border: none; font-size: 14px"
          @click="createClick"
          >注册</el-button
        >
      </div>
      <el-button
        size="normal"
        type="primary"
        style="width: 100%"
        @click="submitLogin"
        >登录</el-button
      >
    </el-form>
    <el-dialog title="用户注册" :visible.sync="dialogFormVisible" width="30%">
      <el-form :model="form" :rules="rule" ref="form">
        <el-form-item label="姓名" :label-width="formLabelWidth" prop="name">
          <el-input v-model="form.name" style="width: 200px"></el-input>
        </el-form-item>
        <el-form-item
          label="用户名"
          :label-width="formLabelWidth"
          prop="username"
        >
          <el-input v-model="form.username" style="width: 200px"></el-input>
        </el-form-item>
        <el-form-item
          label="密码"
          :label-width="formLabelWidth"
          prop="password"
        >
          <el-input
            v-model="form.password"
            style="width: 200px"
            type="password"
          ></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="sureCreate">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "Login",
  data() {
    var nameValidator = (rule, value, callback) => {
      if (this.form.name.length === 0) {
        callback("请输入姓名");
      } else {
        var nameReg = /[\u4e00-\u9fa5]/;
        const numReg = /[0-9]/gi.test(this.form.name);
        const chartReg = /[~#^$@%&!?%.,;:'*]/gi.test(this.form.name);
        const Eng = /[A-z]/g.test(this.form.name);
        if (numReg || chartReg || Eng) {
          callback("姓名只能输入汉字");
        } else {
          callback();
        }
      }
    };
    var usernameValidator = (rule, value, callback) => {
      if (this.form.username.length === 0) {
        callback("请输入用户名");
      } else {
        var usernameReg = /^[A-Za-z0-9]+$/;
        var flag = usernameReg.test(value);
        if (flag) {
          this.getRequest("/employee/basic/checkUsername", {username:this.form.username}).then(
            (resp) => {
              if (resp) {
                callback("用户名已经存在");
              } else {
                callback();
              }
            }
          );
        } else {
          callback("用户名只能输入字母或数字");
        }
      }
    };
    var passwordValidator = (rule, value, callback) => {
      if (this.form.password.length === 0) {
        callback("请输入密码");
      } else if (this.form.password.length > 6) {
        callback("密码最大为6位");
      } else {
        //   const numReg = (/[0-9]/gi).test(this.form.password)
        //   const chartReg = (/[~#^$@%&!?%.,;:'*]/gi).test(this.form.password)
        //   const Eng = (/[A-z]/g).test(this.form.password)
        const Chi = /[\u4e00-\u9fa5]/gi.test(this.form.password);
        if (Chi) {
          callback("密码不能输入汉字");
        } else {
          callback();
        }
      }
    };
    return {
      form: {
        name: "",
        username: "",
        password: "",
      },
      formLabelWidth: "160px",
      dialogFormVisible: false,
      loading: false,
      vcUrl: '/vc.jpg?time='+new Date(),
      loginForm: {
        username: "",
        password: "",
        code: "",
      },
      remeberme:"",
      checked: true,
      rules: {
        username: [
          { required: true, message: "请输入用户名", trigger: "blur" },
        ],
        password: [{ required: true, message: "请输入密码", trigger: "blur" }],
        code: [{ required: true, message: "请输入验证码", trigger: "blur" }],
      },
      rule: {
        name: [{ required: true, validator: nameValidator, trigger: "blur" }],
        username: [
          { required: true, validator: usernameValidator, trigger: "blur" },
        ],
        password: [
          { required: true, validator: passwordValidator, trigger: "blur" },
        ],
      },
    };
  },
  methods: {
    // 点击注册
    createClick() {
      this.form = {
        name: "",
        username: "",
        password: "",
      }
      if (this.$refs["form"] !== undefined) {
        this.$refs["form"].resetFields();
      }
      console.log(this.form)
      this.dialogFormVisible = true;
    },
    // 确认注册
    sureCreate() {
      this.$refs.form.validate((valid) => {
        if (valid) {
          this.postRequest("/employee/basic/addUser", this.form).then(() => {
            this.loading = false;
            this.dialogFormVisible = false;
          });
        }
      });
    },
    updateVerifyCode() {
      this.vcUrl='/vc.jpg?time='+new Date()
    },
    submitLogin() {
      this.$refs.loginForm.validate((valid) => {
        if (valid) {
          if(this.remeberme){
            this.loginForm['remember-me']='on'
          } else {
            delete this.loginForm['remember-me']
          }
          this.loading = true;
          this.postRequest("/doLogin", this.loginForm).then((resp) => {
            this.loading = false;
            if (resp) {
              this.$store.commit("INIT_CURRENTHR", resp.object);
              window.sessionStorage.setItem(
                "user",
                JSON.stringify(resp.object)
              );
              let path = this.$route.query.redirect;
              this.$router.replace(
                path == "/" || path == undefined ? "/home" : path
              );
            } else {
              this.vcUrl = '/vc.jpg?time='+new Date();
            }
          });
        } else {
          return false;
        }
      });
    },
  },
};
</script>

<style>
.loginContainer {
  border-radius: 15px;
  background-clip: padding-box;
  margin: 180px auto;
  width: 350px;
  padding: 15px 35px 15px 35px;
  background: #fff;
  border: 1px solid #eaeaea;
  box-shadow: 0 0 25px #cac6c6;
}

.loginTitle {
  margin: 15px auto 20px auto;
  text-align: center;
  color: #505458;
}

.loginRemember {
  text-align: left;
  margin: 0px 0px 15px 0px;
}
.el-form-item__content {
  display: flex;
  align-items: center;
}
</style>
