<template>
  <view class="login-container">
    <uni-icons type="contact-filled" size="100" color="#afafaf"></uni-icons>
    <button type="primary" class="btn-login" @click="getUserProfile">一键登录</button>
    <text class="tips-text">登录后尽享更多权益</text>
  </view>
</template>

<script>
  import {mapMutations, mapState} from 'vuex'
  
  export default {
    name:"my-login",
    data() {
      return {
        
      };
    },
    computed: {
      ...mapState('m_user', ['redirectInfo']),
    },
    methods: {
      ...mapMutations('m_user', ['updateUserInfo', 'updateToken', 'updateRedirectInfo']),
      async getUserProfile() {
       const res = await uni.getUserProfile({
              desc: '用于完善会员资料', 
            })
        // console.log(res)
        if (res[0] && res[0].errMsg === 'getUserProfile:fail auth deny') return uni.$showMsg('您取消了登录授权！')
        if (res[1] && res[1].errMsg === 'getUserProfile:ok') {
          this.updateUserInfo(res[1].userInfo)
          this.getToken(res[1])
        }
      },
      async getToken(info) {
        const [err, res] = await uni.login().catch(err => err)
        // console.log(res)
        if (err || res.errMsg !== 'login:ok') return uni.$showMsg('登录失败！')
        const query = {
          code: res.code,
          encryptedData: info.encryptedData,
          iv: info.iv,
          rawData: info.rawData,
          signature: info.signature
        }
        
        // console.log(query)
        const {data: loginResult} = await uni.$http.post('/api/public/v1/users/wxlogin', query)
        // console.log(loginResult.meta.status)
        // 测试需要
        if (loginResult.meta.status === 400) {
          this.updateToken('Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1aWQiOjIzLCJpYXQiOjE1NjQ3MzAwNzksImV4cCI6MTAwMTU2NDczMDA3OH0.YPt-XeLnjV-_1ITaXGY2FhxmCe4NvXuRnRB8OMCfnPo')
          this.navigateBack()
          return
        }
        if (loginResult.meta.status !== 200) return uni.$showMsg('登录失败！')
        uni.$showMsg('登录成功！')
        // 由于小程序本身权限问题，属于小程序开发者才能够调用成功登录、支付等比较重要的接口，因为小程序的开发者最多只能90个
      },
      navigateBack() {
        if (this.redirectInfo && this.redirectInfo.openType === 'switchTab') {
          uni.switchTab({
            url: this.redirectInfo.from,
            complete: () => {
              this.updateRedirectInfo(null)
            }
          })
        }
      }
    }
  }
</script>

<style lang="scss">
.login-container {
  height: 750rpx;
  background-color: #f8f8f8;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  position: relative;
  overflow: hidden;
  &::after {
    content: '';
    display: block;
    width: 100%;
    height: 40px;
    background-color: white;
    position: absolute;
    bottom: 0;
    left: 0;
    border-radius: 100%;
    transform: translateY(50%);
  }
  
  .btn-login {
    width: 90%;
    border-radius: 100px;
    margin: 15px 0;
    background-color: #C00000;
  }
  .tips-text {
    font-size: 12px;
    color: gray;
  }
}
</style>