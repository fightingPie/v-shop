<template>
  <div class="loading">正在登录...</div>
</template>
<script>
import { Field, Tab, Tabs, Notify } from 'vant'
import { isPhone, isEmpty } from '@/common/validate'
import util from '@/common/util'
import dayjs from 'dayjs'
var vcodeTimer

export default {
  components: {
    [Field.name]: Field,
    [Tab.name]: Tab,
    [Tabs.name]: Tabs,
    [Notify.name]: Notify
  },
  data() {
    return {
      current: 0, // 0登录,1注册
      mobile: '',
      pwd: '123456',
      nick: '',
      code: '',
      vcodeSend: 0, // 0获取验证码,1验证码倒计时,
      vcodeText: '获取验证码',
      hasLogin: false
    }
  },
  created() {
    const code = this.getQueryString('code')
    this.register(code)
  },
  methods: {
    getQueryString(name) {
      const reg = new RegExp('(^|&)' + name + '=([^&]*)(&|$)')
      var r = window.location.search.substr(1).match(reg)
      if (r != null) return unescape(r[2]); return null
    },
    async register(code) {
      const res = await this.$request.post('/user/wxmp/register/simple', { code })
      if (res.code !== 0) {
        this.$toast(res.msg)
        return
      }
      let _domian = 'http://' + document.domain + '/login'
      _domian = encodeURIComponent(_domian)
      this.$request.get('/site/statistics').then(res => {
        parent.location.href = 'https://open.weixin.qq.com/connect/oauth2/authorize?appid=' + res.data.wxMpAppid + '&redirect_uri=' + _domian + '&response_type=code&scope=snsapi_userinfo&state=STATE#wechat_redirect'
      })
    },
    formSubmit() {
      if (!isPhone(this.mobile)) {
        this.$toast('手机号格式错误')
        return
      }
      // 密码123456
      if (isEmpty(this.pwd)) {
        this.$toast('请输入正确的密码')
        return
      }
      // 用户名
      if (this.current && isEmpty(this.nick)) {
        this.$toast('请输入用户名')
        return
      }
      // 短信验证码4位数字
      if (this.current && !/^\d{4}$/.test(this.code)) {
        this.$toast('请输入正确的验证码')
        return
      }
      // 登录中
      this.$toast.loading({
        mask: true,
        message: '登录中...',
        duration: 0
      })
      // 注册&登录...
      this.current ? this.mobileRegister() : this.mobileLogin()
    },
    mobileRegister() {
      const params = {
        mobile: this.mobile,
        nick: this.nick || `用户${dayjs().format('MM-DD')}`,
        pwd: this.pwd,
        code: this.code,
        autoLogin: true // 该参数可以自动完成登录并返回token
      }
      this.$request.post('/user/m/register', params).then(res => {
        if (res.code !== 0) {
          this.$toast(res.msg)
          return
        }
        // this.mobileLogin()
        util.storage.set('token', res.data.token)
        util.storage.set('uid', res.data.uid)
        this.$route.query.redirect ? this.$router.replace({ path: this.$route.query.redirect }) : this.$router.replace({ path: '/home' })
        // this.$toast.clear()
        this.$toast.success({
          message: '注册成功',
          duration: 1500
        })
      })
    },
    mobileLogin() {
      // 固定为0529，小米note3
      const params = {
        deviceId: '0529',
        deviceName: '小米note3',
        mobile: this.mobile,
        pwd: this.pwd
      }
      this.$request.post('/user/m/login', params).then(res => {
        if (res.code !== 0) {
          this.$toast(res.msg)
          return
        }
        util.storage.set('token', res.data.token)
        util.storage.set('uid', res.data.uid)
        this.$route.query.redirect ? this.$router.replace({ path: this.$route.query.redirect }) : this.$router.replace({ path: '/home' })
        // this.$toast.clear()
        this.$toast.success({
          message: '登录成功',
          duration: 1500
        })
      })
    },
    handleCodeSend() {
      if (!isPhone(this.mobile)) {
        this.$toast('手机号格式错误')
        return
      }
      if (this.vcodeSend === 1) {
        return
      }
      clearInterval(vcodeTimer)
      let time = 60
      vcodeTimer = setInterval(() => {
        if (time <= 0) {
          this.vcodeSend = 0
          this.vcodeText = '获取验证码'
        } else {
          this.vcodeSend = 1
          this.vcodeText = `已发送(${time}s)`
          time--
        }
      }, 1000)
      this.$request.get('/verification/sms/get', { mobile: this.mobile }).then(res => {
        this.$toast('验证码发送成功')
        console.log(`/verification/sms/get：${JSON.stringify(res)}`)
      })
    }
  }
}
</script>

<style lang="less" scoped>
.loading {  
  margin-top: 120px;
  text-align:center;
}
.container {
  height: 100vh;
  padding:50px 0 20px 0;
  box-sizing: border-box;
  display:flex;
  flex-direction: column;
  justify-content: space-between;
}
.logo {
  width: 100px;
  height: 100px;
  margin: 0 auto 30px;
  text-align:center;
}
.btn-area {
  // width: 80%;
  margin: 30px auto;
}
.my-tab .van-tab__pane{
  padding:20px 15px;
}
</style>

