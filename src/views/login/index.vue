<template>
  <div class="login-wrap">
    <van-nav-bar title="登录"/>
    <form>
      <van-cell-group>
        <van-field
          v-model="user.mobile"
          required
          clearable
          label="手机号"
          placeholder="请输入手机号"
          v-validate="'required'"
          name="mobile"
          :error-message="errors.first('mobile')"
        />

        <van-field
          v-model="user.code"
          type="password"
          label="密码"
          placeholder="请输入密码"
          v-validate="'required'"
          name="code"
          :error-message="errors.first('code')"
        />
        </van-cell-group>

       <div class="login-btn-box">
        <van-button
          class="login-btn"
          type="info"
          :loading="loginLoading"
          loading-text="登录中..."
          @click.prevent="handleLogin"
        >登录</van-button>
      </div>
    </form>
  </div>
</template>

<script>
import { login } from '@/api/user'
export default {
  name: 'LoginIndex',
  data () {
    return {
      user: {
        mobile: '15702435689',
        code: '246810'
      },
      loginLoading: false,
      myErrors: {
        mobile: '',
        code: ''
      }
    }
  },
  created () {
    this.configFormErrorMessages()
  },
  methods: {
    async handleLogin () {
      try {
        const valid = await this.$validator.validate()
        if (!valid) {
          return
        }
        this.loginLoading = true
        const data = await login(this.user)
        this.$store.commit('setUser', data)
        this.$router.push({
          name: 'home'
        })
      } catch (err) {
        this.$toast.fail('登录失败')
      }
      this.loginLoading = false
    },
    configFormErrorMessages () {
      const dict = {
        custom: {
          mobile: {
            required: '手机号不能为空'
          },
          code: {
            required: '密码不能为空'
          }
        }
      }
      this.$validator.localize('zh_CN', dict)
    }
  }
}
</script>

<style lang="less" scoped>
.login-btn-box {
  padding: 20px;
  .login-btn {
    width: 100%;
  }
}
</style>
