<template>
  <div>
    <el-form
      v-loading="loading"
      element-loading-text="正在注册..."
      element-loading-spinner="el-icon-loading"
      element-loading-background="rgba(0, 0, 0, 0.8)"
      :rules="rules"
      :model="ruleForm"
      class="registerContainer"
      ref="ruleForm"
    >
      <h1 class="registerTitle">博客注册</h1>
       <el-form-item prop="username">
         <el-input
          type="text"
          placeholder="用户名"
          required="required"
          v-model="ruleForm.username"
          prefix-icon="el-icon-user-solid"
        ></el-input>
         </el-form-item
      >
       <el-form-item prop="telephone">
         <el-input
          class="phone-input"
          placeholder="手机号"
          v-model="ruleForm.telephone"
          prefix-icon="el-icon-mobile-phone"
        ></el-input>
         </el-form-item
      >
       <el-form-item prop="code" v-show="yzmshow">
         <el-input
          v-model="ruleForm.code"
          placeholder="验证码"
          :minlength="6"
          :maxlength="6"
        ></el-input>
         <el-button
          type="primary"
          @click="getCode()"
          class="code-btn"
          :disabled="!show"
        >
           <span v-show="show">发送验证码</span>  <span
            v-show="!show"
            class="count"
            >{{ count }} s</span
          >
           </el-button
        >
         </el-form-item
      >
       <el-form-item prop="password">
         <el-input
          type="password"
          placeholder="请输入密码"
          v-model="ruleForm.password"
          prefix-icon="el-icon-lock"
        ></el-input>
         </el-form-item
      >
       <el-form-item prop="checkPass">
         <el-input
          type="password"
          placeholder="请再次输入密码"
          v-model="ruleForm.checkPass"
          prefix-icon="el-icon-lock"
        ></el-input>
         </el-form-item
      >
       <el-form-item class="btn-form">
         <el-button type="primary" style="width: 100%" @click="submitRegister()"
          >注册</el-button
        >
         <!-- <el-button @click="resetForm('ruleForm')">重置</el-button> -->
         </el-form-item
      >
       </el-form
    >
  </div>
</template>

<script>
export default {
  name: "Register",
  data() {
    var validateUser1 = async (rule, value, callback) => {
      if (value === "") {
        callback(new Error("请输入用户名"));
      } else {
        if (value) {
          this.axios
            .post("/blog/check", {
              username: this.ruleForm.username,
            })
            .then((response) => (this.blogData = response.data));
          if (!this.blogData.obj && this.blogData.obj != null) {
            return callback(new Error("该用户名已经被注册"));
          } else {
            callback();
          }
        }
      }
    };
    var validatePass1 = async (rule, value, callback) => {
      if (value === "") {
        callback(new Error("手机号不能为空"));
      } else {
        const reg = /^1[3|4|5|7|8][0-9]\d{8}$/;
        if (reg.test(value)) {
          this.yzmshow = true;
          callback();
        } else {
          callback(new Error("请输入正确的手机号"));
        }
      }
    };
    var validatePass = (rule, value, callback) => {
      if (value === "") {
        callback(new Error("请输入密码"));
      } else {
        if (this.ruleForm.checkPass !== "") {
          this.$refs.ruleForm.validateField("checkPass");
        }
        callback();
      }
    };
    var validateCode = (rule, value, callback) => {
      if (value === "") {
        callback(new Error("请输入验证码"));
      } else {
        if (this.ruleForm.code.length === 6) {
          callback();
        } else {
          callback(new Error("验证码不正确"));
        }
      }
    };
    var validatePass2 = (rule, value, callback) => {
      if (value === "") {
        callback(new Error("请再次输入密码"));
      } else if (value !== this.ruleForm.password) {
        callback(new Error("两次输入密码不一致!"));
      } else {
        callback();
      }
    };
    return {
      blogData: [],
      loading: false,
      activeIndex: "2",
      loginForm: {
        mobile: "",
        code: "",
        zheCode: "",
      },
      show: true,
      count: "",
      timer: null,
      yzmshow: false,
      ruleForm: {
        username: "",
        pass1: "",
        password: "",
        checkPass: "",
        zhecode: "",
        telephone: "",
        phoneCode: "",
        emailCode: "",
        code: "",
      },
      rules: {
        code: [
          {
            required: true,
            validator: validateCode,
            trigger: "blur",
          },
          {
            min: 6,
            max: 6,
            message: "长度为6",
            trigger: "blur",
          },
        ],
        username: [
          {
            required: true,
            validator: validateUser1,
            trigger: "blur",
          },
        ],
        telephone: [
          {
            required: true,
            validator: validatePass1,
            trigger: "blur",
          },
        ],
        // 密码
        password: [
          {
            required: true,
            validator: validatePass,
            trigger: "blur",
          },
          {
            min: 6,
            message: "长度在不少于6个字符",
            trigger: "blur",
          },
        ],
        // 校验密码
        checkPass: [
          {
            required: true,
            validator: validatePass2,
            trigger: "blur",
          },
          {
            min: 6,
            message: "长度在不少于6个字符",
            trigger: "blur",
          },
        ],
      },
    };
  },
  methods: {
    submitRegister() {
      this.$refs.ruleForm.validate((validate) => {
        // Element自带的校验
        if (validate) {
          this.axios
            .post(
              "/blog/register?username=" +
                this.ruleForm.username +
                "&password=" +
                this.ruleForm.password +
                "&telephone=" +
                this.ruleForm.telephone +
                "&code=" +
                this.ruleForm.code
            )
            .then((resp) => {
              console.log(resp);
               this.$router.replace("/login");
            });
        } else {
          this.$message.error("请输入所有字段");
          return false;
        }
      });
    },
    getCode() {
      this.axios
        .post("/blog/sendVerifyCode?telephone=" + this.ruleForm.telephone)
        .then((resp) => {});
    },
  },
};
</script>

<style scoped>
.registerContainer {
  border-radius: 15px;
  background-clip: padding-box;
  margin: 180px auto;
  width: 350px;
  padding: 15px 35px 15px 35px;
  background: #fff;
  border: 1px solid #eaeaea;
  box-shadow: 0 0 25px #cac6c6;
}
.registerTitle {
  margin: 15px auto 20px auto;
  text-align: center;
  color: #505458;
}
</style>