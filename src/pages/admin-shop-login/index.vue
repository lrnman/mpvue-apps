<template>
  <div class="shop_login">
    <swiper :current="currentTab" class="swiper-box" duration="300" style="height:200rpx" @change="bindChange">
      <swiper-item>
        <div class="options">
          <p>联系电话</p>
          <input type="text" placeholder="请输入您的手机号" maxlength="11" v-model="tel">
        </div>
        <div class="password">
          <p>密码</p>
          <input type="password" placeholder="请输入登录密码" v-model="val">
          <i class="clear" @click="val=''" v-if="val">+</i>
        </div>
      </swiper-item>
      <swiper-item>
        <div class="options">
          <p>联系电话</p>
          <input type="text" placeholder="请输入您的手机号" maxlength="11" v-model="authTel">
        </div>
        <div class="password">
          <p>验证码</p>
          <input type="text" placeholder="请输入验证码" v-model="authVal" maxlength="4">
          <button class="sms_click" plain="true" @click="sendSms" :disabled="countdown>0?true:false" :class="countdown>0?'disabled':''">{{countdownInfo}}</button>
        </div>
      </swiper-item>
    </swiper>
    <div class="btn_login" @click="login">登录</div>
    <div class="login_mode" v-if="currentTab==0">
      <p @click="authLogin" :data-index="1">使用手机验证码登录</p>
      <p @click="signUp">前往注册</p>
    </div>
    <div class="login_mode" v-if="currentTab==1">
      <p @click="passLogin" :data-index="0">使用密码登录</p>
      <p @click="signUp">前往注册</p>
    </div>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        tel: '',
        val: '',
        authTel: '',
        authVal: '',
        currentTab: 0,
        countdown: null,
        countdownInfo: '获取验证码',
        countdownTimer: null,
        userInfo: {},
        info: ''
      }
    },
    onShow() { //页面渲染就会触发
      this.currentTab = 0;
      this.userInfo = wx.getStorageSync('userInfo')
      // console.log(this.userInfo)
      // wx.getStorageSync('token')
    },
    methods: {
      //点击切换
      authLogin(e) {
        // console.log(e.currentTarget.dataset.index)
        if (this.currentTab === e.target.dataset.index) return;
        this.currentTab = e.target.dataset.index;
        this.tel = this.val = '';
      },
      passLogin(e) {
        // console.log(e.currentTarget.dataset.index)
        if (this.currentTab === e.target.dataset.index) return;
        this.currentTab = e.target.dataset.index;
        this.authTel = this.authVal = '';
      },
      //滑动切换
      bindChange(e) {
        // console.log(e)
        this.currentTab = e.target.current;
        this.tel = this.val = this.authTel = this.authVal = '';
      },
      //发送验证码
      sendSms() {
        if (this.phone(this.authTel)) {
          this.util.post({
            url: '/api/Customer/Base/SendSmsCode',
            data: {
              Mobile: this.authTel,
              BizType: 1
            },
            headers: {
              appid: '1',
              // token: 'e6a3823d1e6c4dbe954fe7fbfc4b7140'
            }
          }).then(res => {
            if (res.State == 1) {} else {
              this.msg(res.Msg)
            }
          }).catch(err => {
            console.log(err)
          })
        }
        return;
        this.countdown = 60;
        this.countdownInfo = `${this.countdown}s后重新获取`;
        this.countdownTimer = setInterval(() => {
          this.countdown--;
          this.countdownInfo = `${this.countdown}s后重新获取`;
          if (this.countdown <= 0) {
            clearInterval(this.countdownTimer)
            this.countdownInfo = '重新获取';
          }
        }, 1000)
      },
      login() {
        //判断是密码登录还是验证码登录
        console.log(this.currentTab)
        // return;
        // wx.switchTab({
        //   url: '/pages/admin-index/main'
        // })
        this.util.post({
          url: '/api/Customer/Base/Login',
          data: {
            jsCode: '1',
            qrCodeId: '20180504',
            wxUserInfo: JSON.stringify(this.userInfo)
          },
          headers: {
            appid: '1',
            // token: 'e6a3823d1e6c4dbe954fe7fbfc4b7140'
          }
        }).then(res => {
          if (res.State == 1) {} else {
            this.msg(res.Msg)
          }
        }).catch(err => {
          console.log(err)
        })
      },
      signUp() {
        wx.redirectTo({
          url: '/pages/admin-shop-signup/main'
        })
      },
      //检测手机号
      phone(tel) {
        let reg = /^[1][3,4,5,6,7,8,9]\d{9}$/;
        if (reg.test(tel)) {
          this.info = '';
          return true;
        } else {
          if (tel != '') {
            this.info = '请输入正确的手机号';
            this.msg(this.info)
          } else {
            this.info = '请输入手机号';
            this.msg(this.info)
          }
          return false;
        }
      },
      smsCoding(val) { //短信4位
        let reg = /^\d{4}$/;
        if (reg.test(val)) {
          this.smsInfo = '';
          return true;
        } else {
          if (val != '') {
            this.smsInfo = '请输入完整的短信验证码';
          } else {
            this.smsInfo = '请输入短信验证码';
          }
          return false;
        }
      },
    },
    watch: {
      tel: function(newVal, oldVal) {
        //不是数字的替换为空
        this.tel = newVal.replace(/[^\d]/g, '');
      },
      authTel: function(newVal, oldVal) {
        this.authTel = newVal.replace(/[^\d]/g, '');
      },
      authVal: function(newVal, oldVal) {
        this.authVal = newVal.replace(/[^\d]/g, '');
      },
      currentTab: function(newVal, oldVal) {
        wx.setNavigationBarTitle({
          title: this.currentTab == 0 ? '密码登录' : '验证码登录'
        })
      }
    },
    beforeDestroy() { //清除定时器
      clearInterval(this.countdownTimer)
    },
  }
</script>

<style lang="less">
  .shop_login {
    height: 100%;
    background: #f5f5f5;
    overflow-x: hidden;
    .swiper-box {
      margin-top: 20rpx;
    }
    .options {
      background: #fff;
      display: flex;
      justify-content: flex-start;
      align-items: center;
      padding: 20rpx;
      border-bottom: 1rpx solid #e6e6e6;
      p {
        font-size: 24rpx;
        color: #333;
        width: 140rpx;
        text-align: left;
        transform: translateY(3rpx);
      }
      input {
        flex-grow: 1;
        font-size: 24rpx;
        color: #666;
      }
    }
    .password {
      background: #fff;
      padding: 20rpx;
      display: flex;
      align-items: center;
      justify-content: flex-start;
      position: relative;
      p {
        font-size: 24rpx;
        color: #333;
        width: 140rpx;
        text-align: left;
        transform: translateY(3rpx);
      }
      input {
        flex-grow: 1;
        font-size: 24rpx;
        color: #666;
        padding-right: 20rpx;
      }
      .clear {
        width: 30rpx;
        height: 30rpx;
        border-radius: 50%;
        background: #999;
        color: #666;
        font-size: 28rpx;
        display: flex;
        align-items: center;
        justify-content: center;
        transform: rotate(45deg);
        position: absolute;
        right: 50rpx;
        z-index: 2;
      }
      .sms_click {
        height: 50rpx;
        line-height: 50rpx;
        font-size: 24rpx;
        color: #ff8b03;
        background: transparent;
        border: 1rpx solid #ff8b03;
        border-radius: 6rpx;
        outline: none;
        white-space: nowrap;
        padding: 0 10rpx;
      }
      .disabled {
        color: #999;
        border: 1rpx solid #999;
      }
    }
    .btn_login {
      background: skyblue;
      margin: 20rpx;
      text-align: center;
      height: 80rpx;
      line-height: 80rpx;
      border-radius: 8rpx;
      color: #fff;
      font-size: 30rpx;
    }
    .login_mode {
      padding: 20rpx;
      display: flex;
      align-items: center;
      justify-content: space-between;
      p {
        font-size: 28rpx;
        color: #333;
        text-decoration: underline;
      }
    }
  }
</style>
