<template>
  <el-form
    v-show="getShow"
    ref="formRegist"
    :model="registerData.loginForm"
    :rules="rules"
    class="register-form"
    label-position="top"
    label-width="120px"
    size="large"
  >
  <el-row style="margin-right: -10px; margin-left: -10px">
      <el-col :span="24" style="padding-right: 10px; padding-left: 10px">
        <el-form-item>
          <LoginFormTitle style="width: 100%" />
        </el-form-item>
      </el-col>
      <el-col :span="24" style="padding-right: 10px; padding-left: 10px">
        <el-form-item v-if="registerData.tenantEnable === 'true'" prop="name">
          <el-input
            v-model="registerData.loginForm.name"
            :placeholder="t('login.tenantNamePlaceholder')"
            :prefix-icon="iconHouse"
            link
            type="primary"
            @change="getTenantPackageList()"
          />
        </el-form-item>
      </el-col>

      <el-col :span="24" style="padding-right: 10px; padding-left: 10px">
        <el-form-item prop="packageId">
          <el-select v-model="registerData.loginForm.packageId" clearable placeholder="请选择租户套餐">
            <el-option
              v-for="item in packageList"
              :key="item.id"
              :label="item.name"
              :value="item.id"
            />
          </el-select>
        </el-form-item>
      </el-col>
      <el-col :span="24" style="padding-right: 10px; padding-left: 10px">
        <el-form-item prop="username">
          <el-input
            v-model="registerData.loginForm.username"
            :placeholder="t('login.usernamePlaceholder')"
            :prefix-icon="iconAvatar"
          />
        </el-form-item>
      </el-col>

      <el-col :span="24" style="padding-right: 10px; padding-left: 10px">
        <el-form-item prop="contactName">
          <el-input
            v-model="registerData.loginForm.contactName"
            :placeholder="t('login.contactNamePlaceHolder')"
            :prefix-icon="iconAvatar"
          />
        </el-form-item>
      </el-col>
      <el-col :span="24" style="padding-right: 10px; padding-left: 10px">
        <el-form-item prop="contactMobile">
          <el-input
            v-model="registerData.loginForm.contactMobile"
            :placeholder="t('login.mobileNumberPlaceholder')"
            :prefix-icon="iconMobile"
          />
        </el-form-item>
      </el-col>
      <el-col :span="24" style="padding-right: 10px; padding-left: 10px">
        <el-form-item prop="password">
          <el-input
            v-model="registerData.loginForm.password"
            :placeholder="t('login.passwordPlaceholder')"
            :prefix-icon="iconLock"
            show-password
            type="password"
          />
        </el-form-item>
      </el-col>
      <el-col :span="24" style="padding-right: 10px; padding-left: 10px">
        <el-form-item>
          <XButton
            :loading="loading"
            :title="t('login.btnRegister')"
            class="w-[100%]"
            type="primary"
            @click="getCode()"
          />
        </el-form-item>
      </el-col>
      <Verify
        ref="verify"
        :captchaType="captchaType"
        :imgSize="{ width: '400px', height: '200px' }"
        mode="pop"
        @success="submitForm"
      />
      <el-col :span="24" style="padding-right: 10px; padding-left: 10px">
        <el-form-item>
          <el-row :gutter="5" justify="space-between" style="width: 100%">
            <el-col :span="24">
              <XButton
                :title="t('login.backLogin')"
                class="w-[100%]"
                @click="setLoginState(LoginStateEnum.LOGIN)"
              />
            </el-col>
          </el-row>
        </el-form-item>
      </el-col>
    </el-row>
  </el-form>
</template>
<script lang="ts" setup>
import type { FormRules } from 'element-plus'

import { useValidator } from '@/hooks/web/useValidator'
import LoginFormTitle from './LoginFormTitle.vue'
import { LoginStateEnum, useLoginState } from './useLogin'
import { useIcon } from '@/hooks/web/useIcon'
import * as TenantPackageApi from '@/api/system/tenantPackage'
import { CommonStatusEnum } from '@/utils/constants'
import * as TenantApi from '@/api/system/tenant'

defineOptions({ name: 'RegisterForm' })

const { t } = useI18n()
const { required } = useValidator()
const captchaType = ref('blockPuzzle') // blockPuzzle 滑块 clickWord 点击文字
const getShow = computed(() => unref(getLoginState) === LoginStateEnum.REGISTER)

const registerData = reactive({
  isShowPassword: false,
  captchaEnable: import.meta.env.VITE_APP_CAPTCHA_ENABLE,
  tenantEnable: import.meta.env.VITE_APP_TENANT_ENABLE,
  loginForm: {
    name: '',
    packageId: '',
    contactName: '', // 联系人
    contactMobile: '',
    username: '',
    password: '',
    captchaVerification: '',
    accountCount: 99,
    status: CommonStatusEnum.ENABLE,
    rememberMe: false
  }
})
const verify = ref()
const iconHouse = useIcon({ icon: 'ep:house' })
const iconAvatar = useIcon({ icon: 'ep:avatar' })
const iconLock = useIcon({ icon: 'ep:lock' })
const iconMobile = useIcon({ icon: 'ep:phone'})
const { setLoginState, getLoginState } = useLoginState()
const packageList = ref([] as TenantPackageApi.TenantPackageVO[]) // 租户套餐

// 获取租户 ID
const getTenantPackageList = async () => {
  // 加载套餐列表
  packageList.value = await TenantPackageApi.getTenantPackageList()
}

getTenantPackageList()

const rules: FormRules = {
  tenantName: [required()],
  packageId: [required()],
  username: [required()],
  password: [required()],
  mobileNumber: [required()]
}

const loading = ref(false)

// 获取验证码
const getCode = async () => {
  // 弹出验证码
  verify.value.show()
}

const formRegist = ref()
const message = useMessage() // 消息弹窗
const submitForm = async () => {
  // 校验表单
  if (!formRegist) return
  const valid = await formRegist.value.validate()
  if (!valid) return
  // 提交请求
  formRegist.value = true
  try {
    const data = registerData.loginForm as unknown as TenantApi.TenantVO
    await TenantApi.createTenant(data)
    message.success(t('common.createSuccess'))
    
  } finally {
    setLoginState(LoginStateEnum.LOGIN)
  }
}

</script>
