<template>
  <div v-if="on" class="Login-on">
    <div class="juzhong">
      <input
        type="text"
        v-model="em"
        placeholder="填入你的常用邮箱"
        class="inputbox" />
      <br />
    </div>
    <div class="juzhong">
      <input type="text" v-model="code" placeholder="验证码" class="inputbox" />
      <br />
    </div>
    <span class="WriteInfo">{{ info }}</span>
    <br />
    <div class="juzhong">
      <button :onClick="Send" class="LoginButton">发送验证码</button>
      <button :onClick="Logins" class="LoginButton">登录</button>
    </div>

  </div>
  <div
    :onclick="off"
    v-if="on"
    style="
      z-index: 9;
      position: fixed;
      top: 0;
      left: 0;
      width: 100vw;
      height: 100vh;
      background-color: rgba(0, 0, 0, 0.5);
    "></div>
</template>

<script lang="ts">
import {defineComponent} from 'vue';
import {mapState, mapActions} from 'vuex';
import axios from 'axios';

export default defineComponent({
    computed: {
        ...mapState({
            // 从 Vuex 获取登录状态
            loginx: (state) => state.loginx,
            // 判断是否已经登录
            isLoggedIn: (state) => !!state.loginx.token,
        }),
    },
    created() {
        this.Login(); // 页面加载时检查是否有本地登录信息
    },
    data() {
        return {
            em: '',
            code: '',
            info: '',
        };
    },
    methods: {
        ...mapActions(['setLogin', 'clearLogin']), // 映射 Vuex 的 action

        async Login() {
            const savedLogin = localStorage.getItem('Login');
            if (savedLogin) {
                const parsedLogin = JSON.parse(savedLogin);
                try {
                    const res = await axios.get('https://yzm.z2bguoguo.cn/', {
                        headers: {
                            mode: 'login',
                            em: parsedLogin.em,
                            token: parsedLogin.token,
                        },
                    });

                    if (res.data === 'ok') {
                        this.setLogin(parsedLogin); // 使用 Vuex 设置登录信息
                    } else {
                        this.clearLogin(); // 如果登录信息无效，清除本地存储
                    }

                } catch (error) {
                    console.error('登录校验失败', error);
                    this.clearLogin();
                }
            }
        },

        // 登录逻辑
        async Logins() {
            this.info = '正在登陆';
            try {
                const res = await axios.get('https://yzm.z2bguoguo.cn/', {
                    headers: {
                        mode: 'yzm',
                        em: this.em,
                        code: this.code
                    },
                });
                const response = res.data;
                if (response?.con === 'ok') {
                    const loginData = {
                        token: response.token,
                        em: this.em,
                    };
                    console.log("loginDataToken:" + loginData.token)
                    console.log("loginDataEm:" + loginData.em)
                    this.info = '登陆成功';
                    this.em = '';
                    this.code = '';
                    this.$emit('LoginSwich'); // 通知父组件登录状态变更
                } else {
                    this.code = '';
                    this.info = '登陆失败';
                }
            } catch (error) {
                this.info = '登录请求失败，请重试';
                console.error(error);
            }
        },

        // 发送验证码
        async Send() {
            this.info = '正在发送';
            try {
                const res = await axios.put('https://yzm.z2bguoguo.cn/', this.em);
                if (res.data === 'good') {
                    this.info = '发送成功';
                } else {
                    this.info = '发送失败';
                }
            } catch (error) {
                this.info = '发送验证码失败，请重试';
                console.error(error);
            }
        },

        // 检查本地存储的登录信息
        off() {
            this.$emit('LoginSwich');
        },
    },
    name: 'Login',
    props: ['on'],
});
</script>

<style>
.Login-on {
    position: absolute;
    left: 50%;
    transform: translate(-50%, -50%);
    top: 500%;
    border-radius: 8px;
    z-index: 11;
    height: 330%;
    padding: 10px;
    width: var(--Login-w);
    border: 2px black solid;
    backdrop-filter: blur(4px);
    opacity: 0.7;
    background: var(--color-theme);
}

.juzhong {
    display: flex;
    justify-content: center;
}

.inputbox {
    line-height: 30px;
    margin: 10px;
    width: 70%;
    border: 2px black solid;
}

.inputbox::placeholder {
    font-size: 1.6em;
}

.LoginButton {
    background: transparent;
    border: 2px solid #0099cc;
    border-radius: 6px;
    color: black;
    padding: 8px 16px;
    text-align: center;
    display: inline-block;
    font-size: 16px;
    margin-right: 5%;
    -webkit-transition-duration: 0.4s; /* Safari */
    transition-duration: 0.4s;
    cursor: pointer;
    text-decoration: none;
    text-transform: uppercase;
}

.LoginButton:hover {
    background: #0099cc;
}

.WriteInfo {
    color: #ffcc00;
}
</style>
