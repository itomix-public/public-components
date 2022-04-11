<template>
  <div class="logo-section">
    <van-row align="center">
      <van-col span="8">
      </van-col>
      <van-col span="8" style="text-align: center; margin-top: 5vh;">
        <van-image round width="10vh" height="10vh" :src="logo_image" />
        <p style="font-size: 2.5vh; margin-top: 1vh;">登&nbsp;录</p>
      </van-col>
      <van-col span="8">
      </van-col>
    </van-row>
  </div>
  <div class="login-section">
    <van-config-provider :theme-vars="themeVars">
      <van-form ref="form" @submit="onSubmit">
        <van-cell-group inset style="border: 1px solid #EAEAEA;">
          <van-field
            v-if="use_widget"
            v-model="phone"
            name="phone"
            label="手机号"
            placeholder="手机号"
            readonly clickable
            @touchstart.stop="showKeyboard"
          />
          <van-field
            v-else
            v-model="phone"
            name="phone"
            label="手机号"
            placeholder="手机号"
            :rules="[{ validator, message: '请输入正确手机号' }]"
          />
          <van-field
            v-model="code"
            center
            clearable
            name="code"
            label="短信验证码"
            placeholder="请输入短信验证码"
            :formatter="formatter"
            :rules="[{ required: true, message: '请填写验证码' }]"
          >
            <template #button>
              <van-button @click="sendCode" size="small" type="primary" :disabled="disabled">
                <span v-if="countDown === 60">发送验证码</span>
                <span v-else>{{ countDown }} 秒重新发送</span>
              </van-button>
            </template>
          </van-field>
        </van-cell-group>
        <div class="btn" @click="submit">
          登&nbsp;录
        </div>
      </van-form>
    </van-config-provider>
  </div>

  <van-number-keyboard
    v-model="phone"
    :show="keyboard_show"
    :maxlength="11"
    @blur="onBlur"
  />
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { Toast } from 'vant'
import qs from 'Qs'
import { useRoute, useRouter } from 'vue-router'
import axios from '@/utils/axios'
import logo_image from '@images/logo@3x.png'
import { wxInfo } from '@/utils/tools';
const $route = useRoute();
const $router = useRouter();

  // 判断是否显示控件
  let use_widget = ref(true);
  /**
   * 手机号码校验
   * 函数返回 true 表示校验通过，false 表示不通过
   * @param {*} val 
   */
  const validator = (val) => {
    let flag = false;
    // 简单判断手机号位数
    if (/1\d{10}/.test(val) && phone.value.length === 11) {
      disabled.value = false;
      flag = true;
    } else {
      disabled.value = true;
      flag = false;
    }
    return flag
  };
  onMounted(() => {
    /**
     * 判断微信环境看是否弹出控件框
     * 桌面微信直接输入
     * 其他环境弹出输入框
     */
    if (wxInfo().isiOS || wxInfo().isAndroid) {
      use_widget.value = true;
    } else {
      use_widget.value = false;
    }
  })

  // 手机号输入控件控制
  const keyboard_show = ref(false);
  const showKeyboard = () => { // 弹出数字弹框
    keyboard_show.value = true;
  };
  const onBlur = () => { // 数字键盘失焦回调
    keyboard_show.value = false;
    if (phone.value.length === 11) {
      disabled.value = false;
    }
  };

  // 设置发送短信倒计时
  const countDown = ref(60);
  const countDownHandle = () => {
    // 倒计时
    if (countDown.value === 0) {
      countDown.value = 60;
    } else {
      countDown.value--;
      setTimeout(() => {
        countDownHandle()
      }, 1000)
    }
  }
  const sendCode = () => { // 发送验证码
    if (countDown.value === 60) {
      axios.post('/srv/?f=shzl_comment&a=bind_phone&t=get_code',
      qs.stringify({
        phone: phone.value
      }))
      .then(res => {
        if (res.data.code === 1) {
          Toast.success('发送成功');
          countDownHandle()
        } else {
          console.warn(res.data);
          Toast({
            message: res.data.msg,
            icon: 'close',
          });
        }
      })
      .catch(err => {
        console.error(err); 
      })
    }
  };

  const phone = ref('');
  const code = ref('');
  const disabled = ref(true);
  // 过滤输入的数字 只能四位
  const formatter = (value) => value.substring(0, 4);

  /**
   * 用户登录
   * 3个汪  15500000000  投稿人
   * 点评人1  10000000001  点评人1职业
   * 点评人2  10000000002  点评热2职业
   * 点评人3  10000000003  点评人3职业
   * @param {*} values 
   */

  const onSubmit = (values) => {
    axios.post('/srv/?f=shzl_comment&a=bind_phone&t=bind',
    qs.stringify({
      phone: values.phone,
      code: values.code,
    }))
    .then(res => {
      if (res.data.code === 1) {
        $router.push({
          path: '/'
        });
      } else {
        console.warn(res.data);
        Toast({
          message: res.data.msg,
          icon: 'close',
        });
      }
    })
    .catch(err => {
      console.error(err);
    })
  };

  const themeVars = {
    buttonPrimaryBackground: '#FFFFFF',
    buttonPrimaryBorderColor: '#3FC9E9',
    buttonPrimaryColor: '#3FC9E9',
  };
</script>

<script>
import mixin from 'common/mixin';

export default {
  mixins: [mixin.init],
  data () {
    return {

    }
  },
  mounted () {
    
  },
  methods: {
    submit () {
      let valid = this.$refs.form.validate();
      valid
      .then(() => {
        this.$refs.form.submit();
      })
      .catch(error => {
        console.error(error);
        this.$toast({
          message: '请检查后再次提交',
          icon: 'cross',
        });
      })
    }
  },
}
</script>

<style lang="less" scoped>
  body {
    --van-button-primary-background: white;
  }

  .logo-section {
    height: 22vh;
    // padding: 2vh;
    background-size: cover;
    background-image: url(../assets/images/top-bg.png);
  }

  .login-section {
    background-color: #FAFAFA;
    .btn {
      margin: 16px; 
      background-color: #3FC9E9; 
      text-align: center; 
      color: white; 
      font-size: 2vh;
      padding: 1vh;
      border-radius: 0.5vh;
    }
  }
</style>