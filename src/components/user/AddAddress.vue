<template>
  <div>
    <div class="container">
      <form id="formValidator">
        <div class="address-list">
          <div class="input-line">
            <label>收货地址：</label>
            <span @click="openAddress">{{address.isChecked  ? address.checked.name : address.checkedText}}</span>
          </div>
          <div class='err-class' v-if="validator.detailArea.errIsShow">{{validator.detailArea.errText}}</div>
          <div class="input-line">
            <label for="name">门牌号：</label>
            <input type="text" name="name" placeholder="门牌/楼道等信息" v-model="formData.doorplate">
          </div>
          <div class="input-line">
            <label for="name">联系人：</label>
            <input type="text" name="name" placeholder="您的姓名" v-model="formData.name" @focus="focusMethod('name')" @blur="blurMethod('name')">
          </div>
          <div class='err-class' v-if="validator.name.errIsShow">{{validator.name.errText}}</div>
          <div class="input-line">
            <label for="tel-phone">联系电话：</label>
            <input type="number" name="tel-phone" placeholder="配送员联系您的方式" v-model="formData.tel_phone" @focus="focusMethod('tel_phone')"
              @blur="blurMethod('tel_phone')">
          </div>
          <div class='err-class' v-if="validator.telPhone.errIsShow">{{validator.telPhone.errText}}</div>
          <div class="default">
            <label @click="isDefault()">
              <div class="selected" :class="{'selected1': flag.isDefault}"></div>
              <span>设为默认</span>
            </label>
          </div>
        </div>
        <div class="submit-list">
          <button class="btn-submit" type="button" @click="saveMethod()">提交</button>
        </div>
      </form>
      <!--地址选择弹窗-->
      <div class="address-modal">
        <modal :show="address.show" v-on:close="addressClose">
          <div slot="header">
            <div class="store-header">
              选取地址
            </div>
          </div>
          <div slot="body">
            <div class="input-address">
              <input type="" name="searchAddress" value="" id='searchAddress' placeholder="输入小区，学校，建筑物等" v-model="address.searchKeyword"
                @input="searchAddress()">
            </div>
            <div class="searchAddress">
              <div class="address search-address">
                <p class="title">搜索地址</p>
                <ul>
                  <li class="list" @click="changeArea(item)" v-for="item in address.search">{{item.name}}<span class="area">{{item.adname}}</span></li>
                </ul>
              </div>
            </div>
          </div>
        </modal>
      </div>
    </div>
  </div>
</template>
<script>
  import Vue from 'vue'
  import {
    Swipe,
    SwipeItem,
    MessageBox,
    Toast
  } from 'mint-ui'
  import ajax from '../../utils/ajax.js'
  import Modal from '../common/Modal'
  import utils from '../../utils/public'
  export default {
    name: 'AddAddress',
    components: {
      Swipe,
      SwipeItem,
      MessageBox,
      Modal,
      Toast
    },
    mounted() {
      this.isLoginMethod() //判断登录，并获取列表数据
    },
    data() {
      return {
        isLogin: this.$store.state.user.userInfo.isLogin || '', //是否登录
        userId: this.$store.state.user.userInfo.userId || '', //当前用户ID
        thisId: this.$route.params.id || '',
        listData: '',
        listCity: [],
        listCounty: [],
        address: {
          show: false,
          checked: {},
          isChecked: false,
          checkedText: '请选择收货地址',
          searchKeyword: '',
          search: [],
        },
        flag: {
          isCity: false,
          isCounty: false,
          isDefault: true,
        },
        formData: {
          name: '',
          tel_phone: '',
          detail_area: '',
          district: '',
          doorplate: '',
          default: 1,
          location: '',
        },
        validator: {
          name: {
            errText: '姓名不能为空',
            errIsShow: false,
            isFlag: false
          },
          telPhone: {
            errText: '手机号码不能为空',
            errIsShow: false,
            isFlag: false
          },
          detailArea: {
            errText: '详细地址 不能为空',
            errIsShow: false,
            isFlag: false
          }
        }
      }
    },
    vuerify: {
      name: 'required'
    },
    methods: {
      //  判断是否登录
      isLoginMethod() {
        if (this.isLogin) {
          this.loadShow = true;
          this.getListCity();
          this.getListCounty();
          if (this.thisId) {
            this.getListData(this.thisId)
          }
        } else {
          MessageBox.alert('未登录').then(action => {
            location.href = '/#/site/login'
          })
        }
      },
      //  如果是点击编辑按钮进来的，获取数据
      getListData(id) {
        ajax.getDataFromApi({
          url: `/v1/my-address/${id}/edit`
        }, (response) => {
          this.address.checkedText = response.data.body.detail_area
          this.formData.id = response.data.body.id
          this.formData.name = response.data.body.name
          this.formData.tel_phone = response.data.body.tel_phone
          this.formData.doorplate = response.data.body.doorplate
          this.formData.detail_area = response.data.body.detail_area
          this.formData.district = response.data.body.city_name + '市' + response.data.body.county_name
          this.formData.location = response.data.body.location
          this.validator.name.isFlag = true
          this.validator.telPhone.isFlag = true
          this.validator.detailArea.isFlag = true
        })
      },
      //  获取City数据
      getListCity() {
        ajax.getDataFromApi({
          url: '/area/city'
        }, (response) => {
          this.listCity = response.data.body;
          this.flag.isCity = true
        })
      },
      //  获取County数据
      getListCounty() {
        ajax.getDataFromApi({
          url: '/area/county',
        }, (response) => {
          this.listCounty = response.data.body;
          this.flag.isCounty = true
        })
      },
      //  关闭地址弹窗
      addressClose() {
        this.address.show = false
      },
      //  开启地址弹窗
      openAddress() {
        this.address.show = true
      },
      //  搜索定位
      searchAddress() {
        let key = '6ec262982ede339365a6f9d9b5370f1b'
        let keywords = this.address.searchKeyword
        let types = '银行|学校|小区|建筑|公司'
        let city = '重庆'
        Vue.http.get(
          `http://restapi.amap.com/v3/assistant/inputtips?key=${key}&keywords=${keywords}&types=${types}&city=${city}`
        )
          .then(response => {
            this.address.search = response.data.tips
          })
      },
      //  改变定位
      changeArea(address) {
        this.address.isChecked = true
        this.address.checked = JSON.parse(JSON.stringify(address))
        this.validator.detailArea.isFlag = true
        this.validator.detailArea.errIsShow = false
        this.formData.district = this.address.checked.district
        this.formData.detail_area = this.address.checked.name
        this.formData.location = this.address.checked.location
        this.addressClose()
      },
      //  设为默认
      isDefault(event) {
        this.flag.isDefault = !this.flag.isDefault
        if (this.flag.isDefault) {
          this.formData.default = 1
        } else {
          this.formData.default = 0
        }
      },
      //  提交按钮
      saveMethod() {
        if (this.validator.name.isFlag && this.validator.telPhone.isFlag && this.validator.detailArea.isFlag) {
          if (this.thisId) {
            //  编辑地址，保存
            ajax.putDataToApi({
              url: `/v1/my-address/${this.formData.id}`,
              body: this.formData
            }, (response) => {
              ajax.getDataFromApi({
                url: `/v1/my-address/${this.formData.id}`
              }, response => {
                let address = response.data.body
                address.name = address.detail_area
                this.getNaberStore(address.location)
                utils.sessionstorageData('checkedMyAddress', address)
                utils.sessionstorageData('checkedAddress', address)
                if(utils.sessionstorageGetData('editAddress')){
                    location.href ='/#/site/my-address'
                } else {
                  if (utils.localstorageGetData('buyWay') !== 'cart') {
                  location.href = `/#/site/order-submit/${utils.localstorageGetData('orderId')}`
                } else {
                  location.href = `/#/site/order-submit`
                }
                }
              })
            })
          } else {
            //  新增地址，保存
            ajax.postDataToApi({
              url: '/v1/my-address',
              body: this.formData,
            }, (response) => {
               Toast({
                  message: '地址添加成功',
                  position: 'bottom',
                  duration: 2000
                });
                location.href = '/#/site/my-address'
            })
          }
        } else {
          if (!this.validator.name.isFlag) {
            this.validator.name.errIsShow = true
          }
          if (!this.validator.telPhone.isFlag) {
            this.validator.telPhone.errIsShow = true
          }
          if (!this.validator.detailArea.isFlag) {
            this.validator.detailArea.errIsShow = true
          }
        }
      },
      //    获取云图附件门店
      getNaberStore(location) {
        let key = '6ec262982ede339365a6f9d9b5370f1b'
        let tableid = '586b5c10afdf520ea8f2368e'
        let center = location
        let radius = 1000
        this.$jsonp('http://yuntuapi.amap.com/datasearch/around', {
          key: key,
          tableid: tableid,
          center: center,
          radius: radius
        })
          .then(response => {
            if (response.datas.length === 0) {
              const self = this
              this.naberStore = []
              utils.sessionstorageData('allNavberStore', [])
              utils.sessionstorageData('naberStore',[])
              MessageBox.confirm('您所定位的地址没有推荐门店信息，您可以通过更改定位地址来获取门店商品信息', '门店推荐提示', { confirmButtonText: '换个地址' }).then(action => {
                self.openAddress()
              })
              utils.sessionstorageData('editAddressIsInvaild',false)
            } else {
              if (response.datas && response.datas.length > 0) {
                this.naberStore = response.datas
                utils.sessionstorageData('allNavberStore', response.datas)
                utils.sessionstorageData('naberStore',response.datas[0])
              }
              utils.sessionstorageData('editAddressIsInvaild',true)
            }
          }, err => {
            console.log(err)
          })
      },
      //  验证focus
      focusMethod(currentObj) {
        switch (currentObj) {
          case 'name':
            this.validator.name.errIsShow = false
            break;
          case 'tel_phone':
            this.validator.telPhone.errIsShow = false
            break;
          case 'detail_area':
            this.validator.detailArea.errIsShow = false
            break;
        }
      },
      blurMethod(currentObj) {
        switch (currentObj) {
          case 'name':
            let formDataName = this.formData.name
            let validatorName = this.validator.name
            let nameRe = /^[a-zA-Z\u4e00-\u9fa5]{2,16}$/

            if (formDataName === '') {
              validatorName.errText = '姓名不能为空'
              validatorName.errIsShow = true
              validatorName.isFlag = false
            } else if (!nameRe.test(formDataName)) {
              validatorName.errText = '请输入4到16为姓名'
              validatorName.errIsShow = true
              validatorName.isFlag = false
            } else {
              validatorName.isFlag = true
            }
            break;
          case 'tel_phone':
            let telRe = /^1[3|4|5|8]\d{9}$/
            let formDataTel = this.formData.tel_phone
            let validatorTel = this.validator.telPhone

            if (formDataTel === '') {
              validatorTel.errText = '手机号码不能为空'
              validatorTel.errIsShow = true
              validatorTel.isFlag = false
            } else if (!telRe.test(formDataTel)) {
              validatorTel.errText = '请输入正确的手机号码'
              validatorTel.errIsShow = true
              validatorTel.isFlag = false
            } else {
              validatorTel.isFlag = true
            }
            break;
          case 'detail_area':
            let formDataArea = this.formData.detail_area
            let validatorArea = this.validator.detailArea
            if (formDataArea === '') {
              validatorArea.errText = '请填写详细地址'
              validatorArea.errIsShow = true
              validatorArea.isFlag = false
            } else {
              validatorArea.isFlag = true
            }
            break;
        }
      },
    },
  }
  require('../../assets/scss/addAddress.scss')

</script>