<template>
  <div class="issue-wrap">
    <login-btn ></login-btn>
  	<i-notice-bar icon="systemprompt" closable>
	    {{notice}}
	</i-notice-bar>
	<form @submit="formSubmit" report-submit>
		<div class="upload" :class="goods.icon.length > 0 ? 'has-upload' : 'no-upload'">
			<div class="add-icon" @click="chooseIcon">
				<i-icon type="camera" size="30" color="#333" />
			    <div>上传图片</div>
			</div>
			<scroll-view scroll-x class="icon-wrap" v-if="goods.icon.length > 0">
			<block v-for="(item, index) in goods.icon" :key="index">
				<div class="icon-box">
					<div class="close" @click="delIcon(index)">
						<i-icon type="close" size="12" color="#fff"/>
					</div>
					<div class="default" v-if="index === 0">封面</div>
					<image :src="item" alt="" @click="selectDefault(index)"></image>
				</div>
			</block>
			</scroll-view>
			</div>
	    <div class="space-h"></div>
	    <div class="goods-info">
	    	<div class="goods-name">
	    	    <i-input style="width: 80%" :value="goods.name" @change="changeName" name="name" placeholder="为二货取个诱人的名字吧~" :maxlength="maxlength"/>
	    	    <span class="max-length">{{odd}}</span>
	    	</div>
	    	<div class="goods-price">
	    		<i-input  :value="goods.price" @change="changePrice" name="price" type="number"title="现价" maxlength="10"/>
	    		<i-input :value="goods.oprice" @change="changeOprice" name="oprice" type="number"title="原价" maxlength="10"></i-input>
	    	</div>
	    	<textarea class="goods-detail" name="detail" :class="{hide:(isShowClassify || isShowSchool)}" :value="goods.detail" @input="changeDetail" placeholder="在此描述你的二货。如：品牌，规格，成色等信息。" />
	    	<div class="goods-detail"  :class="{hide:(!isShowClassify && !isShowSchool)}" >
            <span v-if="goods.detail">{{goods.detail}}</span>
            <span v-else class="textarea-placeholder">在此描述你的二货。如：品牌，规格，成色等信息。</span>
  	    </div>
      </div>
	    <div class="space-h"></div>
	    <i-cell-group i-class="border-b">
		    <i-cell title="分类" is-link :value="goods.classify ? goods.classify.name : '选择分类'" @iclick="showClassify"></i-cell>
		    <i-cell title="学校" is-link :value="goods.school ? goods.school.name : '选择学校'" @iclick="showSchool"></i-cell>
		    <!-- <i-cell title="价格" is-link :value="goods.price ? goods.price : '开价'" only-tap-footer="true"></i-cell> -->
		</i-cell-group>
		<i-input i-class="border-b" v-if="goods.school" :value="goods.address" @change="changeAddress" name="address" placeholder="详细地址，如：北校区X区X栋101" maxlength="50"></i-input>
    <i-input :value="goods.number" type="number" @change="changeNumber" name="number" placeholder="输入你的联系方式" maxlength="11"></i-input>
	    <div class="issue fixed-footer">
	      <i-button long="true" type="success" >{{issue ? '发布' : '保存'}}</i-button>
	      <!-- i-button 还未支持formType="submit" 用button做临时处理覆盖到i-button上-->
	      <button formType="submit" style="height: 88rpx;position: absolute; top: 0;left: 0;right: 0;opacity: 0"></button>
	    </div>
	</form>
	<i-action-sheet :visible="isShowClassify" show-cancel @cancel="closeClassify" @iclick="changeClassify" :actions="classify"></i-action-sheet>
	<i-action-sheet :visible="isShowSchool" show-cancel @cancel="closeSchool" @iclick="changeSchool">
    <view slot="header" style="color: #333;padding: 32rpx;">
        选择学校
    </view>
		<!-- 数据多后使用 -->
		<i-index slot="content" >
			<i-index-item v-for="(item, index) in school" :key="index" :name="item.key">
				<div v-for="(s, i) in item.list" :key="i" @click="changeSchool(s)" class="school-item">
					{{s.name}}
				</div>
			</i-index-item>
		</i-index>
	</i-action-sheet>
	<i-toast id="toast" />
	<i-message id="message" />
  </div>
</template>

<script>
import {getCloudPath} from '@/utils/index.js'
import WxValidate from '../../../static/utils/validate.js'
import { $Toast, $Message } from '../../../static/iview/base/index'
import loginBtn from '@/components/loginBtn'
export default {
  components: {
    loginBtn
  },
  data () {
    return {
      notice: '禁止发布虚假信息和危害社会安全信息。',
      issue: false,
      goods: {
        name: '',
        uid: '',
        price: null,
        oprice: null,
        icon: [],
        detail: '',
        classify: null,
        school: null,
        address: '',
        number: null
      },
      classify: [],
      school: [],
      maxlength: 20,
      isShowClassify: false,
      isShowSchool: false,
      isShowPrice: false,
      loading: false,
      rules: {
        name: {
          required: true,
          maxlength: 20
        },
        price: {
          required: true,
          number: true
        },
        oprice: {
          required: true,
          number: true
        },
        detail: {
          required: true
        },
        classify: {
          required: true
        },
        school: {
          required: true
        },
        address: {
          required: true
        },
        icon: {
          required: true
        },
        number: {
          required: true,
          tel: true
        }
      },
      msgs: {
        name: {
          required: '请输入二货名称',
          maxlength: '二货名称不得超过20'
        },
        price: {
          required: '请输入现价',
          number: '请输入数字'
        },
        oprice: {
          required: '请输入原价',
          number: '请输入数字'
        },
        icon: {
          required: '请上传二货图片'
        },
        classify: {
          required: '请选择分类'
        },
        detail: {
          required: '好的描述更容易卖出去哦~'
        },
        school: {
          required: '请选择学校'
        },
        address: {
          required: '请输入详细地址'
        },
        number: {
          required: '请留下联系方式哦',
          tel: '手机号码不对'
        }
      },
      validate: null
    }
  },
  computed: {
    odd () {
      return this.maxlength - this.goods.name.length
    },
    userInfo () {
      return wx.getStorageSync('userInfo')
    }
  },
  mounted () {
    this.validate = new WxValidate(this.rules, this.msgs)
  },
  onLoad (options) {
    if (options.id) {
      wx.setNavigationBarTitle({
        title: '编辑'
      })
      this.issue = false
      this.getGoods(options.id)
    } else {
      this.issue = true
      wx.setNavigationBarTitle({
        title: '发布'
      })
    }
  },
  onReady () {
    if (this.issue) {
      this.goods.school = this.userInfo.school
      this.goods.address = this.userInfo.address
      this.goods.number = this.userInfo.number
    }
  },
  onHide () {
    this.issue && wx.setStorageSync('goods', this.goods)
  },
  onUnload () {
    if (!this.issue) {
      this.goods = {
        name: '',
        uid: '',
        price: null,
        oprice: null,
        icon: [],
        detail: '',
        classify: null,
        school: null,
        address: '',
        number: null
      }
    }
  },
  onShow () {
    if (wx.getStorageSync('goods') && this.issue) {
      this.goods = wx.getStorageSync('goods')
    }
  },
  methods: {
    getGoods (id) {
      wx.showLoading({
        title: '加载中...'
      })
      this.fetch('getOneGood', {
        _id: id
      }).then(res => {
        wx.hideLoading()
        this.goods = res.result
        console.log('goods:', this.goods)
      }).catch(e => {
        wx.hideLoading()
        console.error(e)
      })
    },
    chooseIcon () {
      let that = this
      wx.chooseImage({
        success (res) {
          if (res.errMsg === 'chooseImage:ok') {
            let cloudPath = getCloudPath('goods')
            console.log(res.tempFilePaths, cloudPath)
            wx.cloud.uploadFile({
              cloudPath,
              filePath: res.tempFilePaths[0]
            }).then(r => {
              that.goods.icon.push(r.fileID)
            }).catch(e => {
              console.log(e)
            })
            // wx.cloud.callFunction({
            //   // 云函数名称
            //   name: 'upload',
            //   data: {
            //     cloudPath,
            //     fileContent: res.tempFilePaths
            //   }
            // }).then(r => {
            //   console.log(r)
            //   that.goods.icon.push(r.result.fileID)
            // }).catch(res => {
            //   console.log(res)
            // })
          }
        },
        error (err) {
          console.log(err)
        }
      })
    },
    delIcon (index) {
      let {icon} = JSON.parse(JSON.stringify(this.goods))
      let fileIDs = icon.splice(index, 1)
      wx.cloud.callFunction({
        name: 'delUpload',
        data: {
          fileIDs
        }
      }).then(res => {
        console.log(res)
        this.goods.icon = icon
      })
    },
    selectDefault (index) {
      let {icon} = JSON.parse(JSON.stringify(this.goods))
      this.goods.icon = [...icon.splice(index, 1), ...icon]
    },
    changeName ({target: {detail: {value}}}) {
      this.goods.name = value
    },
    changeDetail ({target: {value}}) {
      this.goods.detail = value
    },
    changePrice ({target: {detail: {value}}}) {
      this.goods.price = value
    },
    changeOprice ({target: {detail: {value}}}) {
      this.goods.oprice = value
    },
    showClassify () {
      if (this.classify.length > 0) {
        this.isShowClassify = true
      } else {
        this.fetch('getClassify').then(res => {
          this.isShowClassify = true
          this.classify = res.result.data
        }).catch(res => {
          console.log('error' + res)
        })
      }
    },
    closeClassify () {
      this.isShowClassify = false
    },
    changeClassify ({target: {index}}) {
      this.goods.classify = this.classify[index]
      this.closeClassify()
    },
    showSchool () {
      console.log(this.school)
      if (this.school.length > 0) {
        this.isShowSchool = true
      } else {
        this.fetch('getSchool').then(res => {
          console.log(res)
          this.isShowSchool = true
          this.school = res.result.data
          console.log(this.school)
        }).catch(res => {
          console.log('error' + res)
        })
      }
    },
    closeSchool () {
      this.isShowSchool = false
    },
    changeSchool (school) {
      this.goods.school = school
      this.closeSchool()
    },
    changeAddress ({target: {detail: {value}}}) {
      this.goods.address = value
    },
    changeNumber ({target: {detail: {value}}}) {
      this.goods.number = value
    },
    formSubmit (e) {
      let data = this.goods
      console.log(e, data)
      // 传入表单数据，调用验证方法
      if (!this.validate.checkForm(data)) {
        const error = this.validate.errorList[0]
        $Toast({
          content: error.msg
        })
      } else {
        this.fetch('setGoods', data).then(res => {
          console.log(res)
          if (res.result.errMsg === 'collection.add:ok') {
            $Message({
              content: '发布成功！',
              type: 'success'
            })
            this.goods = {
              name: '',
              uid: '',
              price: null,
              oprice: null,
              icon: [],
              detail: '',
              classify: null,
              school: null,
              address: ''
            }
            wx.setStorageSync('refresh', true)
            wx.switchTab({
              url: '/pages/index/main'
            })
          } else if (res.result.errMsg === 'collection.update:ok') {
            wx.navigateBack({
              delta: 1,
              success () {
                wx.showToast({
                  title: '保存成功'
                })
                console.log('sss')
                $Message({
                  content: '保存成功！',
                  type: 'success'
                })
              }
            })
          } else {
            $Message({
              content: '发布失败，请重新发布！',
              type: 'error'
            })
          }
        })
      }
    }
  }
}
</script>

<style>
.login-btn{
  position:absolute;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
  opacity: 0;
  z-index: 10000;
}
.issue-wrap{
}
.issue-wrap .spin{

}
.upload{
  height: 200rpx;
	padding: 30rpx;
	box-sizing: border-box;
	display: flex;
	font-size: 28rpx;
	color: #333;
}
.upload.no-upload{
	align-items: center;
	justify-content: center;
}
.upload.has-upload{
	overflow: hidden;
}
.upload.has-upload .icon-box, .upload .add-icon{
	width: 120rpx;
	height: 120rpx;
}
.upload.has-upload .icon-box{
	display: inline-block;
	position: relative;
	margin-left: 20rpx;
}
.upload.has-upload .icon-box>image{
	width: 100%;
	height: 100%;
}
.upload.has-upload .icon-box>.close{
	position: absolute;
	right: 0;
	top: 0;
	display: flex;
	justify-content: center;
	align-items: center;
	width: 26rpx;
	height: 26rpx;
	background: rgba(0, 0, 0, 0.6);
}
.upload.has-upload .icon-box>.default{
	width: 100%;
	height: 30rpx;
	position: absolute;
	bottom: 0;
	display: flex;
	justify-content: center;
	align-items: center;
	color: #fff;
	background: #f76a24;
	opacity: 0.6;
	font-size: 26rpx;
}
.upload .add-icon{
	width: 150rpx;
	display: flex;
	flex-direction: column;
	align-items: center;
	box-sizing: border-box;
	padding-top: 16rpx;
}
.upload>.icon-wrap{
	flex-grow: 1;
}
.upload.has-upload .add-icon{
	/*background: #eee;*/
}
.goods-name{
	display: flex;
	align-items: center;
	position: relative;
	border-bottom: 1rpx solid #eee;
}
.goods-name>.max-length{
	position: absolute;
	right: 30rpx;
	font-size: 26rpx;
	color: #666;
}
.goods-price{
	display: flex;
}
.goods-detail{
	width: 100%;
	height: 200rpx;
	border: none;
	box-sizing: border-box;
	padding: 30rpx;
	font-size: 28rpx;
	line-height:30rpx;
}
.textarea-placeholder{
	font-size: 28rpx;
  color: grey;
  line-height: 28rpx;
}
.school-item{
	padding: 20rpx 40rpx;
}
</style>
