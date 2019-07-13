<template>
  <div class="login">
    <form>
      <van-nav-bar title="登录" />
      <van-cell-group>
        <van-field v-model="user.mobile" required clearable label="手机号" placeholder="请输入手机号" />

        <van-field v-model="user.code" label="验证码" placeholder="请输入验证码" required>
          <van-button slot="button" size="small" type="primary">发送验证码</van-button>
        </van-field>
      </van-cell-group>
      <van-button type="info" @click="handleLogin" :loading="loginLoading" loading-text="登录中...">登录</van-button>
    </form>
  </div>
</template>

<script>
import { login } from '@/api/user'
export default {
  name: 'Login',
  data () {
    return {
      user: {
        mobile: '17777318254',
        code: '246810'
      },
      loginLoading: false
    }
  },
  methods: {
    async handleLogin () {
      this.loginLoading = true
      try {
        const data = await login(this.user)
        this.$store.commit('setUser', data)
        this.loginLoading = false
        this.$router.push({
          name: 'home'
        })
      } catch (err) {
        console.log(err)
        this.loginLoading = false
        this.$toast.fail('登录失败！')
      }
    }
  }
}
</script>

<style lang="less" scoped>
.login {
  width: 375px;
  .van-button--info {
    margin: 10px;
    width: 355px;
  }
}
</style>
