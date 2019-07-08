<template>
  <div class="login">
    <form>
      <van-nav-bar title="登录"  />
      <van-cell-group>
        <van-field v-model="user.mobile" required clearable label="手机号" placeholder="请输入手机号" />

        <van-field v-model="user.code" label="验证码" placeholder="请输入验证码" required>
          <van-button slot="button" size="small" type="primary">发送验证码</van-button>
        </van-field>
      </van-cell-group>
      <van-button type="info" @click="handleLogin">登录</van-button>
    </form>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  name: 'Login',
  data () {
    return {
      user: {
        mobile: '17777318254',
        code: '123456'
      }
    }
  },
  methods: {
    async handleLogin () {
      const res = await axios({
        method: 'POST',
        url: 'http://toutiao.course.itcast.cn/app/v1_0/authorizations',
        data: this.user
      })
      try {
        console.log(res)
      } catch (err) {
        console.log(err)
      }
    }
  }
}
</script>

<style lang="less" scoped>
.login {
  width: 375px;
  .van-button--info{
      margin: 10px;
      width: 355px;
  }
}
</style>
