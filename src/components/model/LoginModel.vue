<template>
  <v-dialog v-model="loginFlag" :fullscreen="isMobile" max-width="460">
    <v-card class="login-container" style="border-radius:4px">
      <v-icon class="float-right" @click="loginFlag = false">
        mdi-close
      </v-icon>
      <div class="login-wrapper">
        <!-- 用户名 -->
        <v-text-field
          v-model="username"
          label="邮箱号"
          placeholder="请输入您的邮箱号"
          clearable
          @keyup.enter="login"
        />
        <!-- 密码 -->
        <v-text-field
          v-model="password"
          class="mt-7"
          label="密码"
          placeholder="请输入您的密码"
          @keyup.enter="login"
          :append-icon="show ? 'mdi-eye' : 'mdi-eye-off'"
          :type="show ? 'text' : 'password'"
          @click:append="show = !show"
        />
        <!-- 按钮 -->
        <v-btn
          class="mt-7"
          block
          color="blue"
          style="color:#fff"
          @click="login"
        >
          登录
        </v-btn>
        <!-- 注册和找回密码 -->
        <div class="mt-10 login-tip">
          <span @click="openRegister">立即注册</span>
          <span @click="openForget" class="float-right">忘记密码?</span>
        </div>
      </div>
    </v-card>
  </v-dialog>
</template>

<script>
export default {
  data: function() {
    return {
      username: "",
      password: "",
      show: false
    };
  },
  computed: {
    loginFlag: {
      set(value) {
        this.$store.state.loginFlag = value;
      },
      get() {
        return this.$store.state.loginFlag;
      }
    },
    isMobile() {
      const clientWidth = document.documentElement.clientWidth;
      return clientWidth <= 960;
    },
    socialLoginList() {
      return this.$store.state.blogInfo.websiteConfig.socialLoginList;
    },
    showLogin() {
      return function(type) {
        return this.socialLoginList.indexOf(type) !== -1;
      };
    }
  },
  methods: {
    openRegister() {
      this.$store.state.loginFlag = false;
      this.$store.state.registerFlag = true;
    },
    openForget() {
      this.$store.state.loginFlag = false;
      this.$store.state.forgetFlag = true;
    },
    login() {
      var reg = /^[A-Za-z0-9\u4e00-\u9fa5]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+$/;
      if (!reg.test(this.username)) {
        this.$toast({ type: "error", message: "邮箱格式不正确" });
        return false;
      }
      if (this.password.trim().length === 0) {
        this.$toast({ type: "error", message: "密码不能为空" });
        return false;
      }
      const that = this;
      // eslint-disable-next-line no-undef
      var captcha = new TencentCaptcha(this.config.TENCENT_CAPTCHA, function(
        res
      ) {
        if (res.ret === 0) {
          //发送登录请求
          let param = new URLSearchParams();
          param.append("username", that.username);
          param.append("password", that.password);
          that.axios.post("/api/login", param).then(({ data }) => {
            if (data.flag) {
              that.username = "";
              that.password = "";
              that.$store.commit("login", data.data);
              that.$store.commit("closeModel");
              that.$toast({ type: "success", message: "登录成功" });
            } else {
              that.$toast({ type: "error", message: data.message });
            }
          });
        }
      });
      // 显示验证码
      captcha.show();
    }
  }
};
</script>

<style scoped>
.social-login-wrapper a {
  text-decoration: none;
}
</style>
