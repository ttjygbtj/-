<script>
import Vue from 'vue'

export default {
  onLaunch: function () {
    console.log('App Launch');
    // #ifdef APP-PLUS
    plus.push.getClientInfoAsync(
        info => {
          console.log('Success', JSON.stringify(info.clientid));
        },
        e => {
          console.log('Failed', JSON.stringify(e));
        }
    );
    // #endif

    uni.$on('sign_in', userId => {
      console.log('监听到事件来自登陆页面登入记录，userId' + userId);
      if (userId) {
        this.connectSocket(userId);
      }
    });
    try {
      const token = uni.getStorageSync('token');
      this.$store.commit('SetToken', token);
      if (token) {
        this.$store.commit('SetToken', token);
        this.getUserInfo(token);
      } else {
        uni.reLaunch({
          url: '/pages/index/index'
          // url: '/pages/login/login'
        });
      }
    } catch (e) {
      uni.reLaunch({
        url: '/pages/index/index'
        // url: '/pages/login/login'
      });
    }
    uni.getSystemInfo({
      success: function (e) {
        // #ifndef MP
        Vue.prototype.StatusBar = e.statusBarHeight;
        if (e.platform == 'android') {
          Vue.prototype.CustomBar = e.statusBarHeight + 50;
        } else {
          Vue.prototype.CustomBar = e.statusBarHeight + 45;
        }
        ;
        // #endif

        // #ifdef MP-WEIXIN
        Vue.prototype.StatusBar = e.statusBarHeight;
        let custom = wx.getMenuButtonBoundingClientRect();
        Vue.prototype.Custom = custom;
        Vue.prototype.CustomBar = custom.bottom + custom.top - e.statusBarHeight + 4;
        // #endif

        // #ifdef MP-ALIPAY
        Vue.prototype.StatusBar = e.statusBarHeight;
        Vue.prototype.CustomBar = e.statusBarHeight + e.titleBarHeight;
        // #endif
      }
    });
  },
  onShow: function () {
    console.log('App Show')
  },
  onHide: function () {
    console.log('App Hide')
  }, onUnload() {
    uni.closeSocket();
  },
  methods: {
    getUserInfo() {
      getUserInfo()
          .then(data => {
            this.$store.commit('SetUser', data.user);
            this.connectSocket(data.user.id);
            uni.reLaunch({
              url: '/pages/index/index'
            });
          })
          .catch(err => {
            uni.reLaunch({
              url: '/pages/index/index'
              // url: '/pages/login/login'
            });
          });
    },
    connectSocket(userId) {
      uni.connectSocket({
        url: SOCKET_URL + `/webSocket/` + userId,
        success: res => {
          uni.onSocketOpen(() => {
            console.log('WebSocket连接已打开！');
          });
          uni.onSocketMessage(result => {
            console.log('服务器接收内容');
            let data = JSON.parse(result.data);
            console.log(data);
            if (typeof data.msg !== 'undefined') {
              this.$store.commit('SetMessages', data.msg);
            }
            this.$store.commit('SetUserChatList', data.userList);
          });
          uni.onSocketError(err => {
            uni.showToast({
              title: 'WebSocket连接发生错误！',
              icon: 'none',
              duration: 2000
            });
            console.log(err);
          });
          uni.onSocketClose(close => {
            uni.showToast({
              title: 'webSocket 已关闭！',
              icon: 'none',
              duration: 2000
            });
            console.log('WebSocket 已关闭！');
          });
        },
        fail: () => {
          uni.showToast({
            title: '连接失败，请检查网络',
            icon: 'none',
            duration: 2000
          });
        }
      });
    }
  }
}
</script>

<style lang="scss">
@import "/static/iconfont/font.scss";
@import "colorui/main.css";
@import "colorui/icon.css";
@import "uview-ui/index.scss";
</style>
