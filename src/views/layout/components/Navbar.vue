<template>
  <el-menu class="navbar" mode="horizontal">
    <!--<hamburger class="hamburger-container" :toggleClick="toggleSideBar" :isActive="sidebar.opened"></hamburger>-->

    <!--<breadcrumb class="breadcrumb-container"></breadcrumb>-->
    <div style="line-height: 50px;
    height: 50px;
    float: left;
    padding: 0 10px;">
      <el-menu
          class="el-top-menu"
          mode="horizontal"
          background-color="#545c64"
          text-color="#fff"
          active-text-color="#ffd04b">
        <template v-for="item in permission_routers">
          <router-link v-if="!item.hidden&&item.children&&item.children.length===1"
                       :to="{ path: item.path+'/'+item.children[0].path, query: item.children[0].query}"
                       :key="item.children[0].name">
            <el-menu-item :index="item.path+'/'+item.children[0].path">
              <span v-if="item.children[0].meta&&item.children[0].meta.title">{{item.children[0].meta.title}}</span>
            </el-menu-item>
          </router-link>

          <el-submenu :show-timeout="100" v-if="!item.hidden&&item.children&&item.children.length>1"
                      :index="item.name||item.path" :key="item.name">
            <template slot="title">
              <span v-if="item.meta&&item.meta.title">{{item.meta.title}}</span>
            </template>

            <template v-if="!child.hidden" v-for="child in item.children">
              <!--<sidebar-item class="nest-menu" v-if="child.children&&child.children.length>0" :routes="[child]" :key="child.path"></sidebar-item>-->

              <router-link :to="{ path: item.path+'/'+child.path, query: child.query}" :key="child.name">
              <el-menu-item :index="item.path+'/'+child.path">
                <span v-if="child.meta&&child.meta.title">{{child.meta.title}}</span>
              </el-menu-item>
              </router-link>
            </template>
          </el-submenu>
        </template>
      </el-menu>
    </div>
    <div class="right-menu">
      <el-dropdown trigger="click" v-if="group_id != 10">
            <span class="el-dropdown-link" @click="getInitData">
              <el-tag type="warning">
                <span style="color: #fff">账户余额：</span><span style="color: #F56C6C">¥ {{asset}}</span>
              </el-tag>
            </span>
        <el-dropdown-menu slot="dropdown">
          <el-dropdown-item divided>
            <span style="display:block;">账户余额：{{asset}}</span>
          </el-dropdown-item>
          <el-dropdown-item divided>
            <span style="display:block;">冻结金额：{{frozen_balance}}</span>
          </el-dropdown-item>
          <el-dropdown-item divided>
            <span style="display:block;">可提现金额：{{balance}}</span>
          </el-dropdown-item>
        </el-dropdown-menu>
      </el-dropdown>
      <span style="font-size: 14px;margin-left: 20px;">
          商户号：<span style="color:#F56C6C">{{user.user.main_merchant_id}}</span>
      </span>
      <el-dropdown class="avatar-container right-menu-item">
        <div class="avatar-wrapper">
          欢迎您:
          <span style="color:#F56C6C">{{nickname_dis}}</span>
          <i class="el-icon-caret-bottom"></i>
          <audio src="/static/mp3/6005.mp3" controls="controls" preload id="remind" hidden></audio>
        </div>
        <el-dropdown-menu slot="dropdown">
          <el-dropdown-item divided>
            <span @click="editPassDialog" style="display:block;">修改密码</span>
          </el-dropdown-item>
          <el-dropdown-item divided v-if="group_id != 10">
            <span @click="showHandleAuthKeyDialog" v-if="user.user.main_merchant_id == user.user.id" style="display:block;">查看/修改商户KEY</span>
          </el-dropdown-item>
          <el-dropdown-item divided>
            <span @click="getGoogleCode" style="display:block;">安全令牌</span>
          </el-dropdown-item>
          <el-dropdown-item divided v-if="group_id != 10">
            <span v-if="user.user.main_merchant_id == user.user.id" @click="handleFinancial" style="display:block;">设置/修改资金密码</span>
          </el-dropdown-item>
          <el-dropdown-item divided>
            <span @click="handleBindLoginIp" style="display:block;">绑定登录IP</span>
          </el-dropdown-item>
            <el-dropdown-item divided v-if="group_id == 30">
                <span @click="handleBindIp" style="display:block;" v-if="user.user.main_merchant_id == user.user.id">绑定API接口IP地址</span>
            </el-dropdown-item>
          <el-dropdown-item divided v-if="group_id == 30">
            <span @click="emailVisible = true" style="display:block;" v-if="user.user.main_merchant_id == user.user.id">变更邮箱</span>
          </el-dropdown-item>
          <el-dropdown-item divided v-if="group_id == 30">
            <span @click="clearGoogleVisible = true" style="display:block;" v-if="user.user.main_merchant_id == user.user.id">解绑安全令牌</span>
          </el-dropdown-item>
          <el-dropdown-item divided v-if="group_id == 30">
            <span @click="clearFinancialVisible = true" style="display:block;" v-if="user.user.main_merchant_id == user.user.id">清空资金密码</span>
          </el-dropdown-item>
          <el-dropdown-item divided>
            <span @click="logout" style="display:block;">退出登录</span>
          </el-dropdown-item>
        </el-dropdown-menu>
      </el-dropdown>
    </div>
    <el-dialog title="修改密码" :visible.sync="editPassVisible" width="30%">
      <el-form :model="editPassForm">
        <dd style="color: red;">提示：密码必须包含一个大写字母，一个小字母，一个数字；长度（6-16）</dd>
        <el-form-item label="旧密码：" label-width="120px">
          <el-input size="small" type="password" v-model="editPassForm.oldPass" style="width: 200px"></el-input>
        </el-form-item>
        <el-form-item label="新密码：" label-width="120px">
          <el-input size="small" type="password" v-model="editPassForm.newPass" style="width: 200px"></el-input>
        </el-form-item>
        <el-form-item label="确认密码：" label-width="120px">
          <el-input size="small" type="password" v-model="editPassForm.confirmPass" style="width: 200px"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button size="small" @click="editPassVisible = false">取 消</el-button>
        <el-button size="small" type="primary" @click="editPassHandle">提交</el-button>
      </div>
    </el-dialog>
    <el-dialog title="绑定安全令牌" :visible.sync="googleCodeVisible" width="50%">
      <el-form :model="editPassForm">
        <div>
                  <span style="float: left;width: 200px;height: 260px">
                      <!--<img :src="googleCode" />-->
                      <vue-qr :text="googleCode" height="200" width="200"></vue-qr>
                      <el-input size="small" v-model="key_2fa" style="width: 180px"></el-input>
                  </span>
          <span style="height: 260px">
                      <p style="height: 20px;">1. 造访 App Store</p>
                      <p style="height: 20px;">2. 搜寻 「Google Authenticator」 或 「谷歌身份验证器」</p>
                      <p style="height: 20px;">3. 下载并安装应用程式</p>
                      <p style="height: 20px;">4. 用谷歌身份验证器扫描二维码（QR CODE）</p>
                      <p style="height: 20px;">5. 输入谷歌身份验证器产生的验证码</p>
                      <p style="height: 20px;">6. 提交后将本账号绑定谷歌身份验证器</p>
                      <p style="height: 20px;">7. 绑定后，本账号登录时，必须输入验证器产生的验证码，方可登录</p>
                  </span>
        </div>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button size="small" @click="googleCodeVisible = false">取 消</el-button>
        <el-button size="small" type="primary" @click="setGoogleCode">提交</el-button>
      </div>
    </el-dialog>
    <el-dialog title="修改商户KEY" :visible.sync="authKeyVisible" width="30%">
      <el-form>
        <el-form-item label="商户KEY：" label-width="120px">
          <el-input size="small" type="textarea" :rows="3" v-model="auth_key" style="width: 200px"></el-input>
          <dd style="color: red;line-height: 20px;margin-left: 0;">提示：商户Key值长度不能超过36位</dd>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button size="small" @click="authKeyVisible = false">取 消</el-button>
        <el-button size="small" type="primary" @click="editAuthKey">提交</el-button>
      </div>
    </el-dialog>
    <el-dialog title="设置/修改资金密码" :visible.sync="editFinancialPassVisible" width="30%">
      <el-form :model="editFinancialPassForm">
        <dd style="color: red;">提示：密码必须包含一个大写字母，一个小字母，一个数字；长度（6-16）</dd>
        <el-form-item label="旧资金密码：" label-width="120px" v-if="editFinancialPassForm.is_financial == 1">
          <el-input size="small" type="password" v-model="editFinancialPassForm.oldPass"
                    style="width: 200px"></el-input>
        </el-form-item>
        <el-form-item label="资金密码：" label-width="120px">
          <el-input size="small" type="password" v-model="editFinancialPassForm.newPass"
                    style="width: 200px"></el-input>
        </el-form-item>
        <el-form-item label="确认资金密码：" label-width="120px" v-if="editFinancialPassForm.is_financial == 1">
          <el-input size="small" type="password" v-model="editFinancialPassForm.confirmPass"
                    style="width: 200px"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button size="small" @click="editFinancialPassVisible = false">取 消</el-button>
        <el-button size="small" type="primary" @click="editFinancialPassHandle">提交</el-button>
      </div>
    </el-dialog>
    <el-dialog title="验证登录会话安全性" :visible.sync="sessionSecurityCheckDialogVisible" width="30%">
      <el-form :model="sessionSecurityCheckForm">
        <dd style="color: red;">提示：资金密码和安全令牌必须填至少一项</dd>
        <el-form-item label="资金密码：" label-width="120px">
          <el-input size="small" type="password" v-model="sessionSecurityCheckForm.finacialPwd"
                    style="width: 200px"></el-input>
        </el-form-item>
        <el-form-item label="安全令牌：" label-width="120px">
          <el-input size="small" v-model="sessionSecurityCheckForm.key2fa"
                    style="width: 200px"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button size="small" @click="sessionSecurityCheckDialogVisible = false">取 消</el-button>
        <el-button size="small" type="primary" @click="sessionSecurityCheckForm.handle">提交</el-button>
      </div>
    </el-dialog>

    <el-dialog
        title="绑定登录IP"
        :visible.sync="loginIpFormVisible"
        @close="close"
        width="40%">
      <template>
        <el-form :model="loginIpForm">
          <p style="color: red;padding-left: 180px;">提示：请务必先行添加当前登录IP!IP有多个 以英文符号分号; 分隔</p>
          <el-form-item v-if="loginIpForm.bind_login_ip.length > 0" label="已绑定IP：" label-width="180px">
            {{loginIpForm.bind_login_ip}}
          </el-form-item>
          <el-form-item label="登录IP地址：" label-width="180px">
            <el-input size="small" type="textarea" :rows="3" v-model="add_login_ip" style="width: 300px"></el-input>
          </el-form-item>
        </el-form>
      </template>
      <span slot="footer" class="dialog-footer">
                <el-button @click="close">取 消</el-button>
                <el-button type="primary" @click="updateLoginIp">确 定</el-button>
            </span>
    </el-dialog>
    <el-dialog
            title="绑定邮箱"
            :visible.sync="emailVisible"
            @close="close"
            width="40%">
      <template>
        <el-form :model="emailForm">
          <p style="color: red;padding-left: 180px;">提示：请填写能正常使用的邮箱地址</p>
          <el-form-item label="邮箱地址：" label-width="180px">
            <el-input size="small" type="email" v-model="emailForm.email" style="width: 300px"></el-input>
          </el-form-item>
          <p style="padding-left: 180px;"> {{user.user.email}}</p>
          <el-form-item label="邮箱验证码：" label-width="180px">
            <el-input size="small" v-model="emailForm.code" style="width: 200px"></el-input>
            <el-button type="primary" size="small" @click="getEmailCode('updateEmail')">发送邮箱验证码</el-button>
          </el-form-item>
        </el-form>
      </template>
      <span slot="footer" class="dialog-footer">
                <el-button @click="close">取 消</el-button>
                <el-button type="primary" @click="updateMail">确 定</el-button>
            </span>
    </el-dialog>
      <el-dialog
              title="绑定API接口IP"
              :visible.sync="ipVisible"
              width="40%">
          <template>
              <el-form :model="ipForm">
                  <p style="color: red;padding-left: 180px;">提示：API接口IP有多个以英文符号分号;分隔</p>
                  <p style="color: red;padding-left: 180px;">如：127.0.0.1;127.0.0.2</p>
                  <el-form-item label="API接口IP地址：" label-width="180px">
                      <el-input size="small" type="textarea" :rows="3" v-model="ipForm.app_server_ips" style="width: 300px"></el-input>
                  </el-form-item>
                  <p style="padding-left: 180px;"> {{user.user.email}}</p>
                  <el-form-item label="邮箱验证码：" label-width="180px">
                      <el-input size="small" v-model="ipForm.code" style="width: 200px"></el-input>
                      <el-button type="primary" size="small" @click="getEmailCode('bindApiIp')">发送邮箱验证码</el-button>
                  </el-form-item>
              </el-form>
          </template>
          <span slot="footer" class="dialog-footer">
                <el-button @click="close">取 消</el-button>
                <el-button type="primary" @click="updateApiIps">确 定</el-button>
            </span>
      </el-dialog>
    <el-dialog
            title="清空安全令牌"
            :visible.sync="clearGoogleVisible"
            width="40%">
      <template>
        <el-form>
          <p style="padding-left: 180px;"> {{user.user.email}}</p>
          <el-form-item label="邮箱验证码：" label-width="180px">
            <el-input size="small" v-model="clearGoogleCode" style="width: 200px"></el-input>
            <el-button type="primary" size="small" @click="getEmailCode('clearGoogle')">发送邮箱验证码</el-button>
          </el-form-item>
        </el-form>
      </template>
      <span slot="footer" class="dialog-footer">
                <el-button @click="close">取 消</el-button>
                <el-button type="primary" @click="updateGoogle">确 定</el-button>
            </span>
    </el-dialog>
    <el-dialog
            title="清空资金密码"
            :visible.sync="clearFinancialVisible"
            width="40%">
      <template>
        <el-form>
          <el-form-item label="安全令牌：" label-width="180px">
            <el-input size="small" v-model="clearFinancialCode" style="width: 200px"></el-input>
          </el-form-item>
        </el-form>
      </template>
      <span slot="footer" class="dialog-footer">
                <el-button @click="close">取 消</el-button>
                <el-button type="primary" @click="updateFinancial">确 定</el-button>
            </span>
    </el-dialog>
  </el-menu>

</template>

<script>
  import {mapGetters} from 'vuex'
  import Breadcrumb from '@/components/Breadcrumb'
  import Hamburger from '@/components/Hamburger'
  import ThemePicker from '@/components/ThemePicker'
  import Screenfull from '@/components/Screenfull'
  // import ErrorLog from '@/components/ErrLog'
  import common from '@/utils/common'

  import axios from '@/utils/http'
  import VueQr from 'vue-qr'
  import ScrollPane from '@/components/ScrollPane'

  export default {
    components: {
      Breadcrumb,
      Hamburger,
      ThemePicker,
      // ErrorLog,
      Screenfull,
      VueQr,
      ScrollPane
    },
    data() {
      return {
        constTrue: true,
        nickname_dis: '',
        constFalse: false,
        group_id: null,
        asset: null,
        frozen_balance: null,
        balance: null,
          main_merchant_id:null,
        // log: errLogStore.state.errLog,
        shoppingCartVisible: false,
        orderFormVisible: false,
        googleCodeVisible: false,
        authKeyVisible: false,
        editFinancialPassVisible: false,
        googleCode: null,
        key_2fa: null,
        auth_key: null,
        old_auth_key: null,
        editPassVisible: false,
        editPassForm: {
          oldPass: null,
          newPass: null,
          confirmPass: null,
        },
        editFinancialPassForm: {
          is_financial: null,
          oldPass: null,
          newPass: null,
          confirmPass: null,
        },
        sessionSecurityCheckDialogVisible: false,
        sessionSecurityCheckForm: {
          finacialPwd: '',
          key2fa: '',
          handle: function () {

          },
        },
        activeIndex: '1',
        activeIndex2: '1',
        loginIpFormVisible:false,
        loginIpForm:{
          bind_login_ip:''
        },
          add_login_ip:'',
          emailForm:{
            email:null,
              code:null,
              type:null,
          },
          // email:null,
          emailVisible:false,
          ipVisible: false,
          ipForm: {
              app_server_ips: null,
              channel_account_id:null,
              code: null
          },
          clearGoogleVisible:false,
          clearGoogleCode:null,
          clearFinancialVisible:false,
          clearFinancialCode:null,
      }
    },
    computed: {
      ...mapGetters([
        'permission_routers',
        'sidebar',
        'nickname',
        'username',
        'user',
        'avatar',
        'language',
      ]),
    },
    created() {
      this.getInitData()
      this.nickname_dis = this.nickname
      if (this.nickname_dis == '') this.nickname_dis = this.username

      let siteInfo = common.getStorage('siteInfo')
      if (typeof siteInfo.siteName != 'undefined') {
        document.title = siteInfo.siteName
      }
    },
    mounted() {
        // this.email = this.user.user.email
      //设置定时器，每30秒刷新一次
      setInterval(this.checkRemitStatus,10 * 1000)
      //资金等检测
      setInterval(this.getInitData, 10 * 1000)
    },
    methods: {
      handleTopMenuSelect(key, keyPath) {
        console.log(key, keyPath);
      },
      getInitData() {
          if(!common.getToken() || typeof this.user.user.from_profile == "undefined" || this.user.user.from_profile!='1'){
            return
          }
          if(this.group_id != 10){
              let self = this
              axios.post('/user/user-check').then(
                  res => {
                      if (res.code == 0) {
                          self.editFinancialPassForm.is_financial = res.data.is_financial;
                          self.group_id = res.data.group_id;
                          self.asset = res.data.asset;
                          self.frozen_balance = res.data.frozen_balance;
                          self.balance = res.data.balance;
                      } else {
                          self.$message.error({message: res.message})
                      }
                  }
              );
          }
      },
      getGoogleCode() {
        let self = this
        axios.post('/user/get-google-code').then(
          res => {
            if (res.code == 0) {
              self.googleCode = res.data;
              self.googleCodeVisible = true;
            } else {
              self.$message.error({message: res.message})
            }
          }
        );
      },
      toggleSideBar() {
        this.$store.dispatch('toggleSideBar')
      },
      handleSetLanguage(lang) {
        this.$i18n.locale = lang
        this.$store.dispatch('setLanguage', lang)
        this.$message({
          message: 'switch language success',
          type: 'success'
        })
      },
      logout() {
        this.$store.dispatch('LogOut').then(() => {
          location.reload()// 为了重新实例化vue-router对象 避免bug
        })
      },
      handleBindLoginIp() {
        this.loginIpFormVisible = true;
        if(this.loginIpForm.bind_login_ip.length == 0){
            this.loginIpForm.bind_login_ip = this.user.user.bind_login_ip == ''?'':JSON.parse(this.user.user.bind_login_ip).join(";")
        }
      },

      updateLoginIp() {
        let self = this
          self.loginIpFormVisible = false;
        if (self.add_login_ip.length < 1) {
          self.$message.error({message: 'ip不能为空'});
          return false;
        }
        if(self.loginIpForm.bind_login_ip.length == 0){
            self.loginIpForm.bind_login_ip = self.add_login_ip
        }else{
            self.loginIpForm.bind_login_ip = self.loginIpForm.bind_login_ip + ";" + self.add_login_ip
        }
        if (self.loginIpForm.bind_login_ip.length > 0) {
          var tmpIp = self.loginIpForm.bind_login_ip.split(';');
          let regIp = /^(\d{1,2}|1\d\d|2[0-4]\d|25[0-5])\.(\d{1,2}|1\d\d|2[0-4]\d|25[0-5])\.(\d{1,2}|1\d\d|2[0-4]\d|25[0-5])\.(\d{1,2}|1\d\d|2[0-4]\d|25[0-5])$/;
          for (let i = 0; i < tmpIp.length; i++) {
            if (!regIp.test(tmpIp[i])) {
              self.$message.error({message: tmpIp[i]+' IP地址格式不正确，请检查'});
              return false;
            }
          }
        }
        let data = {
          ip: tmpIp,
        }
        axios.post('/user/bind-login-ip', data).then(
          res => {
              self.close()
            if (res.code != 0) {
              self.$message.error({message: res.message})
            } else {
              self.$message.success({message: '绑定成功'});
              self.getInitData()
            }
          },
        )
      },
        close(){
            this.add_login_ip = ''
            this.loginIpFormVisible = false
            this.emailVisible = false
            this.emailForm = {
                email:null,
                code:null,
                type:null
            }
            this.ipVisible = false
            this.ipForm = {
                channel_account_id: null,
                code: null
            }
            this.clearGoogleCode = null
            this.clearGoogleVisible = false
            this.clearFinancialVisible = false
            this.clearFinancialCode = null
        },
      editPassHandle() {
        let self = this
        let reg = /^.*(?=.{6,16})(?=.*\d)(?=.*[A-Z])(?=.*[a-z]).*$/;
        if (!reg.test(self.editPassForm.newPass) || !reg.test(self.editPassForm.confirmPass)) {
          self.$message.error({message: '密码格式错误'})
          return false;
        }
        if (self.editPassForm.newPass != self.editPassForm.confirmPass) {
          self.$message.error({message: '新密码与确认密码不一致'})
          return false;
        }
        let data = {
          oldPass: self.editPassForm.oldPass,
          newPass: self.editPassForm.newPass,
          confirmPass: self.editPassForm.confirmPass,
        }
        axios.post('/user/edit-pass', data).then(
          res => {
            if (res.code == 0) {
              self.$message.success({message: '密码修改成功'})
              self.editPassVisible = false;
            } else {
              self.$message.error({message: res.message})
            }
          }
        );
      },
      editPassDialog() {
        this.editPassVisible = true;
        this.editPassForm = {
          oldPass: null,
          newPass: null,
          confirmPass: null,
        }
      },
      setGoogleCode() {
        let self = this
        let data = {
          key_2fa: self.key_2fa
        }
        axios.post('/user/set-google-code', data).then(
          res => {
            self.key_2fa = null;
            if (res.code == 0) {
              self.$message.success({message: '设置成功'})
              self.googleCodeVisible = false;
            } else {
              self.$message.error({message: res.message})
            }
          }
        );
      },
      showHandleAuthKeyDialog() {
          if(this.user.user.main_merchant_id != this.user.user.id){
              this.$message.error({message: '您没有此操作权限'})
              return false
          }
        this.sessionSecurityCheckDialogVisible = true
        this.sessionSecurityCheckForm.handle = this.handleAuthKey
      },
      handleAuthKey() {
        let self = this

        //合并会话验证字段
        let data = {
          finacialPwd: this.sessionSecurityCheckForm.finacialPwd,
          key2fa: this.sessionSecurityCheckForm.key2fa,
        }

        axios.post('/user/get-auth-key', data).then(
          res => {
            if (res.code == 0) {
              self.auth_key = res.data;
              self.old_auth_key = res.data;
              self.authKeyVisible = true;
              self.sessionSecurityCheckDialogVisible = false
            } else {
              self.$message.error({message: res.message})
            }
          }
        );
      },
      editAuthKey() {
        let self = this
        if (self.auth_key.length > 36) {
          self.$message.error({message: '商户Key值长度超过规定长度'})
          return
        }
        if (self.auth_key == self.old_auth_key) {
          this.$message.error({message: '商户Key值没有做任何修改'})
          this.authKeyVisible = false;
          return
        }
        let data = {
          authKey: self.auth_key
        }
        axios.post('/user/edit-auth-key', data).then(
          res => {
            if (res.code == 0) {
              self.authKeyVisible = false;
              self.$message.success({message: '商户Key值修改成功'})
            } else {
              self.$message.error({message: '验证码获取错误'})
            }
          }
        );
      },
      handleFinancial() {
        this.editFinancialPassVisible = true;
      },
      editFinancialPassHandle() {
        let self = this
        let data = {};
        let reg = /^.*(?=.{6,16})(?=.*\d)(?=.*[A-Z])(?=.*[a-z]).*$/;

        if (self.editFinancialPassForm.is_financial == 1) {
          if (!reg.test(self.editFinancialPassForm.newPass) || !reg.test(self.editFinancialPassForm.confirmPass)) {
            self.$message.error({message: '密码格式错误'})
            return false;
          }
          if (self.editFinancialPassForm.newPass != self.editFinancialPassForm.confirmPass) {
            self.$message.error({message: '新密码与确认密码不一致'})
            return false;
          }
          data = {
            oldPass: self.editFinancialPassForm.oldPass,
            newPass: self.editFinancialPassForm.newPass,
            confirmPass: self.editFinancialPassForm.confirmPass,
          }
        } else {
          if (!reg.test(self.editFinancialPassForm.newPass)) {
            self.$message.error({message: '密码格式错误'})
            return false;
          }
          data = {
            newPass: self.editFinancialPassForm.newPass,
          }
        }
        axios.post('/user/edit-financial-pass', data).then(
          res => {
            if (res.code == 0) {
              self.$message.success({message: '密码修改成功'})
              self.editFinancialPassVisible = false;
            } else {
              self.$message.error({message: res.message})
            }
          }
        );
      },
      checkRemitStatus() {
        if(typeof this.user.user.from_profile == "undefined" || this.user.user.from_profile!='1'){
          return
        }
        if (this.group_id == 10) {
          axios.post('/admin/remit/remind').then(
            res => {
              if (res.code == 0 && res.data.count > 0) {
                let audio = document.getElementById('remind')
                if (audio !== null) {
                  audio.currentTime = 0;
                  audio.volume = 1;
                  audio.play();
                }
              }
            }
          );
        }
      },
        getEmailCode(type){
            let self = this;
            let data = {
                type:type
            }
            axios.post('/user/email-code',data).then(
                res => {
                    if (res.code == 0 ) {
                        self.$message.success({message:res.message})
                    }else {
                        self.$message.error({message:'发送失败'})
                    }
                }
            );
        },
        updateMail(){
            let self = this;
            let regEmail = /^[a-zA-Z0-9_.-]+@[a-zA-Z0-9-]+(\.[a-zA-Z0-9-]+)*\.[a-zA-Z0-9]{2,6}$/;
            if (self.emailForm.email != null && !regEmail.test(self.emailForm.email)) {
                self.$message.error({message: '邮箱格式错误'})
                return false;
            }
            if(self.emailForm.code == null || self.emailForm.code.lenth == 0){
                self.$message.error({message: '邮箱验证码错误'})
                return false;
            }
            self.emailForm.type = 'updateEmail'
            axios.post('/user/update-email',self.emailForm).then(
                res => {
                    self.close()
                    if (res.code == 0 ) {
                        self.$message.success({message:res.message})
                    }else {
                        self.$message.error({message:res.message})
                    }
                }
            );
        },
        handleBindIp() {
            this.ipVisible = true;
            // console.log(this.user.user)
            if(this.ipForm.app_server_ips.length == 0){
                this.ipForm.app_server_ips = this.user.user.app_server_ips;
            }
            this.ipForm.channel_account_id = this.user.user.channel_account_id;
        },
        updateApiIps() {
            let self = this
            let tmpIp = []
            if (self.ipForm.app_server_ips.length > 0) {
                tmpIp = self.ipForm.app_server_ips.split(';');
                let regIp = /^(\d{1,2}|1\d\d|2[0-4]\d|25[0-5])\.(\d{1,2}|1\d\d|2[0-4]\d|25[0-5])\.(\d{1,2}|1\d\d|2[0-4]\d|25[0-5])\.(\d{1,2}|1\d\d|2[0-4]\d|25[0-5])$/;
                for (let i = 0; i < tmpIp.length; i++) {
                    if (!regIp.test(tmpIp[i])) {
                        self.$message.error({message: '有IP地址格式不正确，请检查'});
                        return false;
                    }
                }
            }
            if(self.ipForm.code == null || self.ipForm.code.lenth == 0){
                self.$message.error({message: '邮箱验证码错误'})
                return false;
            }
            let data = {
                app_server_ips: tmpIp,
                channel_account_id:self.ipForm.channel_account_id,
                code:self.ipForm.code,
                type:'bindApiIp'
            }
            axios.post('/user/update-bind-api-ips', data).then(
                res => {
                    self.close()
                    if (res.code != 0) {
                        self.$message.error({message:res.message })
                    } else {
                        self.$message.success({message: res.message});
                    }
                },
            )
        },
        updateGoogle(){
            let self = this;
            if(self.clearGoogleCode == null || self.clearGoogleCode.lenth == 0){
                self.$message.error({message: '邮箱验证码错误'})
                return false;
            }
            axios.post('/user/clear-google',{code:self.clearGoogleCode,type:'clearGoogle'}).then(
                res => {
                    self.close()
                    if (res.code == 0 ) {
                        self.$message.success({message:res.message})
                    }else {
                        self.$message.error({message:res.message})
                    }
                }
            );
        },
        updateFinancial(){
            let self = this;
            if(self.clearFinancialCode == null || self.clearFinancialCode.lenth == 0){
                self.$message.error({message: '安全令牌错误'})
                return false;
            }
            axios.post('/user/clear-financial',{code:self.clearFinancialCode}).then(
                res => {
                    self.close()
                    if (res.code == 0 ) {
                        self.$message.success({message:res.message})
                    }else {
                        self.$message.error({message:res.message})
                    }
                }
            );
        }
    }
  }
</script>
<style rel="stylesheet/scss" lang="scss" scoped>
  .el-submenu__title {
    height: 50px;
  }

  .el-menu--horizontal {
    border-bottom: 0px;
  }
  .navbar {
    height: 50px;
    line-height: 50px;
    border-radius: 0px !important;
    background-color: #545c64;

    .el-top-menu {
      float: left;

      .el-submenu {
        overflow: visible !important;
      }

      .el-submenu__icon-arrow {
        display: inline-block !important;
      }

      .is-opened ul {
        /*display: block;*/
      }

    }

    .nickname {
      display: inline-block;
      font-size: 14px;
      line-height: 50px;
      margin-right: 10px;
      float: left;
      color: #606266;
    }
    .nickname a {
      color: #409EFF;
    }
    .hamburger-container {
      line-height: 58px;
      height: 50px;
      float: left;
      padding: 0 10px;
    }
    .breadcrumb-container {
      float: left;
    }
    .errLog-container {
      display: inline-block;
      vertical-align: top;
    }
    .right-menu {
      float: right;
      height: 100%;
      color: #fff;
      .el-dropdown {
        color: #fff;
      }
      &:focus {
        outline: none;
      }
      .right-menu-item {
        display: inline-block;
        margin: 0 8px;
      }
      .screenfull {
        height: 20px;
      }
      .international {
        vertical-align: top;
        .international-icon {
          font-size: 20px;
          cursor: pointer;
          vertical-align: -5px;
        }
      }
      .theme-switch {
        vertical-align: 15px;
      }
      .avatar-container {
        height: 50px;
        margin-right: 30px;
        .avatar-wrapper {
          cursor: pointer;
          margin-top: 5px;
          position: relative;
          .user-avatar {
            width: 40px;
            height: 40px;
            border-radius: 10px;
          }
          .el-icon-caret-bottom {
            position: absolute;
            right: -20px;
            top: 25px;
            font-size: 12px;
          }
        }
      }
    }
  }
</style>



