<template>
  <div class="login-container">
    <!-- 导航栏 -->
    <van-nav-bar class="page-nav-bar" title="登录" />
    <!-- /导航栏 -->

    <!-- Form 表单 -->
    <van-form ref="loginForm" @submit="onSubmit">
      <!-- 手机号 -->
      <van-field v-model="user.mobile" name="mobile" placeholder="请输入手机号" :rules="userFormRules.mobile" type="number" maxlength="11">
        <i slot="left-icon" class="toutiao toutiao-shouji"></i>
      </van-field>
      <!-- 验证码 -->
      <van-field v-model="user.code" center clearable name="code" placeholder="请输入短信验证码" :rules="userFormRules.code" type="number" maxlength="6">
        <i slot="left-icon" class="toutiao toutiao-yanzhengma"></i>
        <template #button>
          <!-- time 倒计时时间 -->
          <van-count-down v-if="isCountDownShow" :time="1000 * 60" format="ss s" @finish="isCountDownShow = false" />
          <van-button v-else round size="small" type="default" class="send-sms-btn" @click="onSendSms" native-type="button">发送验证码</van-button>
        </template>
      </van-field>
      <!-- 登录按钮 -->
      <div class="login-btn-wrap">
        <van-button class="login-btn" block type="info" native-type="submit">
          登录
        </van-button>
      </div>
    </van-form>
    <!-- /Form 表单 -->
  </div>
</template>

<script>
import { login, sendSms } from '@/api/user.js'

export default {
  data () {
    return {
      user: {
        // 手机号
        mobile: '13911111111',
        // 验证码
        code: '246810'
      },
      // 校验规则
      userFormRules: {
        mobile: [{ required: true, message: '手机号不能为空' }, { pattern: /1[3|5|7|8]\d{9}/ }, { message: '手机号格式错误' }],
        code: [{ required: true, message: '验证码不能为空' }, { pattern: /^\d{6}$/ }, { message: '验证码格式错误' }]
      },
      // 倒计时
      isCountDownShow: false
    }
  },
  created () {},
  mounted () {},
  methods: {
    async onSubmit (values) {
      // 1.获取表单数据
      const user = this.user
      // 2.表单验证
      this.$toast.loading({
        message: '加载中...',
        forbidClick: true,
        duration: 0 // 持续时间，如果为 0 ，则持续展示
      })
      // 3.提交表单请求登录
      try {
        const { data } = await login(user)
        this.$store.commit('setUser', data.data)
        this.$toast.success('登录成功')
      } catch (err) {
        if (err.response.status === 400) {
          this.$toast.fail('手机号或验证码错误')
        } else {
          this.$toast.fail('未知错误，请稍后重试')
        }
      }
      // 4. 根据请求响应结果处理后续操作
    },
    // 发送验证码
    async onSendSms () {
      // 1.校验手机号
      try {
        await this.$refs.loginForm.validate('mobile')
      } catch (err) {
        return console.log('校验失败', err)
      }
      // 2.验证通过，显示倒计时
      this.isCountDownShow = true
      // 3.请求发送验证码
      try {
        await sendSms(this.user.mobile)
        this.$toast.success('发送成功')
      } catch (err) {
        this.isCountDownShow = false
        if (err.response.status === 429) {
          this.$toast('发送太频繁，请稍后重试')
        } else {
          this.$toast('发送失败，请重试')
        }
      }
    }
  }
}
</script>

<style scoped lang="less">
.login-container {
  .toutiao {
    font-size: 37px;
  }
  .send-sms-btn {
    width: 156px;
    height: 46px;
    line-height: 46px;
    background-color: #ededed;
    font-size: 22px;
    color: #666;
  }
  .login-btn-wrap {
    padding: 53px 33px;
  }
  .login-btn {
    background-color: #6db4fb;
    border: none;
  }
}
</style>
