<template>
  <div class="navbar">
    <hamburger :is-active="sidebar.opened" class="hamburger-container" @toggleClick="toggleSideBar" />

    <breadcrumb class="breadcrumb-container" />
    <!-- el-input.input-search(
        placeholder='请输入分类名称'
        v-model='search'
        size="mini"
        clearable
        style="width:230px"
        @clear="doSearch")
        el-button(slot="append" icon="el-icon-search" type="primary" size="mini" @click="doSearch") -->
    <div class="right-menu">
      <div class="avatar-wrapper">
        <span style="margin-right:25px; font-weight:400; color:#606266; font-size:14px">
          {{ userInfo.userName }}
        </span>
        <!-- <i class="el-icon-caret-bottom" /> -->
      </div>
      <!-- <el-dropdown class="avatar-container" trigger="click">
        <div class="avatar-wrapper">
          <span>
            {{ userInfo.userName }}
          </span>
          <i class="el-icon-caret-bottom" />
        </div>
        <el-dropdown-menu slot="dropdown" class="user-dropdown">
          <router-link to="/">
            <el-dropdown-item>
              主页
            </el-dropdown-item>
          </router-link>
          <el-dropdown-item divided>
            <span style="display:block;" @click="showModify">修改密码</span>
          </el-dropdown-item>
          <el-dropdown-item v-if="userInfo.UserType === 'Department'" divided>
            <span style="display:block;" @click="showUpload">上传印章</span>
          </el-dropdown-item>
          <el-dropdown-item divided>
            <span style="display:block;" @click="logout">退出</span>
          </el-dropdown-item>
        </el-dropdown-menu>
      </el-dropdown> -->
    </div>

  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import Breadcrumb from '@/components/Breadcrumb'
import Hamburger from '@/components/Hamburger'

import md5 from 'js-md5'

export default {
  components: {
    Breadcrumb,
    Hamburger
  },
  filters: {
    filterImg: (val) => {
      return process.env.VUE_APP_BASE_API + val
    }
  },
  data() {
    var validatePass = (rule, value, callback) => {
      if (!value) {
        callback(new Error('请输入密码'))
      } else {
        if (this.form.checkPassword !== '') {
          this.$refs.pwdform.validateField('checkPassword')
        }
        callback()
      }
    }
    var validatePass2 = (rule, value, callback) => {
      if (!value) {
        callback(new Error('请再次输入密码'))
      } else if (value !== this.form.newPassword) {
        callback(new Error('两次输入密码不一致!'))
      } else {
        callback()
      }
    }
    // var isPhone = (rule, value, callback) => {
    //   if (value === '') {
    //     callback(new Error('app登录名不能为空'))
    //   } else {
    //     if (!checkPhone(value)) {
    //       callback(new Error('app登录名必须是电话号码'))
    //     } else {
    //       callback()
    //     }
    //   }
    // }
    // var isPhoneX = (rule, value, callback) => {
    //   if (value === '') {
    //     callback(new Error('手机号不能为空'))
    //   } else {
    //     if (!checkPhone(value)) {
    //       callback(new Error('手机号必须是电话号码'))
    //     } else {
    //       callback()
    //     }
    //   }
    // }
    return {
      dialogFormVisible: false,
      form: {},
      oldPasswordType: 'password',
      newPasswordType: 'password',
      checkPasswordType: 'password',
      rules: {
        oldPassword: [{ required: true, message: '请输入旧密码', trigger: 'blur' }],
        newPassword: [{ validator: validatePass, required: true, trigger: 'blur' }],
        checkPassword: [{ validator: validatePass2, required: true, trigger: 'blur' }]
      },
      // 用户弹窗
      dialogUploadVisible: false,
      userRules: {
        Seal: [
          { required: true, message: '公章不能为空', trigger: 'change' }
        ]
      },
      formDialog: {
        visible: false,
        title: '',
        type: 'update'
      },
      infoForm: {},
      imgFile: '',
      search: ''
    }
  },
  computed: {
    ...mapGetters([
      'sidebar',
      'avatar',
      'userInfo'
    ]),
    userLy: function() {
      if (this.userInfo.UserType === 'Admin') {
        return '超级管理员'
      } else if (this.userInfo.UserType === 'Company') {
        const object = this.userInfo.Companys
        const codes = this.userInfo.CompanyCodes
        let companys = ''
        for (const key in object) {
          if (object.hasOwnProperty(key)) {
            const e = object[key]
            const ikey = codes.find(n => n === key)
            if (ikey) {
              companys = e.Name + companys
            }
          }
        }
        return companys
      } else {
        const object = this.userInfo.Departments
        const codes = this.userInfo.DepartmentCodes
        let departments = ''
        for (const key in object) {
          if (object.hasOwnProperty(key)) {
            const e = object[key]
            const ikey = codes.find(n => n === key)
            if (ikey) {
              departments = e.Name + departments
            }
          }
        }
        return departments
      }
    },
    action() {
      return `${process.env.VUE_APP_BASE_API}/Basic/UploadImage`
    }
  },
  methods: {
    handleSealSuccess(res, file) {
      // this.imageUrl = URL.createObjectURL(file.raw)
      const url = res.Data.url
      this.$set(this.infoForm, 'Seal', url)
    },
    // 上传限制
    beforeAvatarUpload(file) {
      const isJPG = file.type === 'image/jpeg'
      const isPNG = file.type === 'image/png'
      const isLt2M = file.size / 1024 / 1024 < 2
      if (!isJPG && !isPNG) {
        this.$message.error('上传图片只能是 JPG 格式!')
      }
      if (!isLt2M) {
        this.$message.error('上传图片大小不能超过 2MB!')
      }
      return (isJPG || isPNG) && isLt2M
    },
    toggleSideBar() {
      this.$store.dispatch('app/toggleSideBar')
    },
    async logout() {
      await this.$store.dispatch('user/logout')
      this.$router.push(`/login?redirect=${this.$route.fullPath}`)
    },
    showModify() {
      this.dialogFormVisible = true
    },
    /**
     * @description: '提交修改密码'
     * @param {type} ''
     * @return: ''
     */

    showPwd(type) {
      if (this[type] === 'password') {
        this[type] = ''
      } else {
        this[type] = 'password'
      }
      this.$nextTick(() => {
        this.$refs[type].focus()
      })
    },
    open(formName) {
      this.$nextTick(() => {
        this.$refs[formName].clearValidate()
        this.form = {}
      })
    },
    /** *** 基本设置 start ******/

    showUpload() {
      this.formDialog.type = 'update'
      this.dialogUploadVisible = true
      this.infoForm.Code = this.userInfo.DepartmentMode.Code
      this.infoForm.PCode = this.userInfo.DepartmentMode.PCode
      this.infoForm.Name = this.userInfo.DepartmentMode.Name
      this.infoForm.XZQHCode = this.userInfo.DepartmentMode.XZQHCode
      // this.infoForm.Seal = this.userInfo.DepartmentMode.Seal
      this.$set(this.infoForm, 'Seal', this.userInfo.DepartmentMode.Seal)
    },

  }
}
</script>

<style lang="scss" scoped>
.navbar {
  height: 50px;
  overflow: hidden;
  position: relative;
  background: #fff;
  box-shadow: 0 1px 4px rgba(0,21,41,.08);

  .hamburger-container {
    line-height: 46px;
    height: 100%;
    float: left;
    cursor: pointer;
    transition: background .3s;
    -webkit-tap-highlight-color:transparent;

    &:hover {
      background: rgba(0, 0, 0, .025)
    }
  }

  .breadcrumb-container {
    float: left;
  }

  .right-menu {
    float: right;
    height: 100%;
    line-height: 24px;
    margin: 13px;

    &:focus {
      outline: none;
    }

    .right-menu-item {
      display: inline-block;
      padding: 0 8px;
      height: 100%;
      font-size: 18px;
      color: #5a5e66;
      vertical-align: text-bottom;

      &.hover-effect {
        cursor: pointer;
        transition: background .3s;

        &:hover {
          background: rgba(0, 0, 0, .025)
        }
      }
    }

    .avatar-container {
      padding-right: 30px;
      cursor: pointer;
      .avatar-wrapper {
        position: relative;
        .user-avatar {
          cursor: pointer;
          width: 40px;
          height: 40px;
          border-radius: 10px;
        }
        .el-icon-caret-bottom {
          cursor: pointer;
          position: absolute;
          right: -20px;
          top: 6px;
          font-size: 12px;
        }
      }
    }
  }
}
.dialog-class{
  /deep/ .el-form-item__content{
    display: flex;

    .show-pwd {
      // position: absolute;
      // right: 10px;
      padding-right: 10px;
      font-size: 16px;
      color: #889aa4;
      cursor: pointer;
      user-select: none;
    }
    // .el-input{
    //   width: 220px
    // }
  }
}
.upload{
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
  display: inline-block;
  text-align: center;
  outline: 0;
  &:hover{
    border-color: #409EFF;
  }
}
.avatar-uploader {
  cursor: pointer;
  /deep/ .el-upload{
    border: 1px dashed #d9d9d9;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
    &:hover{
      border-color: #409EFF;
    }
  }
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
.avatar {
  width: 178px;
  height: 178px;
  display: block;
}
.pt10{
  padding-top: 10px;
}
.input-search{
  height: 50px;
  display: inline-block;
}
</style>
