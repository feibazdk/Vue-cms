<template>
	<div>
		<div class='title'>添加商品信息</div>
		<el-steps :active="active" finish-status="success">
			<el-step title="步骤 1"></el-step>
			<el-step title="步骤 2"></el-step>
			<el-step title="步骤 3"></el-step>
			<el-step title="步骤 4"></el-step>
			<el-step title="步骤 5"></el-step>
		</el-steps>
		<el-form ref="addProductForm" :rules="rules" label-width="80px">
			<el-tabs :value="nowTab" @tab-click="toggleTabs" :tab-position="tabPosition" name="basic">
				<el-tab-pane label="基本信息" name="basic">
					<!-- 商品基本信息 -->
					<el-form-item label="商品名称">
						<el-input v-model='pform.goods_name'></el-input>
					</el-form-item>
					<el-form-item label="商品价格">
						<el-input  v-model='pform.goods_price'></el-input>
					</el-form-item>
					<el-form-item label="商品重量">
						<el-input  v-model='pform.goods_weight'></el-input>
					</el-form-item>
					<el-form-item label="商品数量">
						<el-input  v-model='pform.goods_number'></el-input>
					</el-form-item>
					<el-form-item label="商品分类">
						<el-cascader
						placeholder="请选择分类"
						expand-trigger="hover"
						:change-on-select="false"
						:show-all-levels="false"
						:options="pform.pcategory"
						:props="pform.selfDefineAttr"
						v-model="pform.selectedOptions">
						</el-cascader>
					</el-form-item>
					<el-form-item label="是否促销">
						<el-radio-group v-model="pform.is_promote" size="medium">
						<el-radio border label="true">是</el-radio>
						<el-radio border label="false">否</el-radio>
						</el-radio-group>
					</el-form-item>
				</el-tab-pane>
				<el-tab-pane label="动态参数" name="params">
					<!-- 动态参数 -->
					<el-form-item :key='item.attr_id' v-for='item in pform.dparam' :label="item.attr_name">
						<el-checkbox-group size="medium" v-model='item.attr_vals'>
						<el-checkbox border :key='index' v-for='(tag, index) in item.attr_vals' :label="tag" :name="'p'+item.attr_id"></el-checkbox>
						</el-checkbox-group>
					</el-form-item>
				</el-tab-pane>
				<el-tab-pane label="静态属性" name="attrs">
					<!-- 静态属性 -->
					<el-form-item :key='item.attr_id' v-for='item in pform.sparam' :label="item.attr_name">
						<el-input v-model="item.attr_vals"></el-input>
					</el-form-item>
				</el-tab-pane>
				<el-tab-pane label="图片上传" name="uploader">
					<el-upload
						:action="baseUrl"
						:headers='token'
						list-type="picture"
						multiple
						:on-preview="handlePictureCardPreview"
						:on-success="handleSuccess"
						:on-remove="handleRemove">
						<el-button type="success" plain>上传<i class="el-icon-upload el-icon--right"></i></el-button>
					</el-upload>
					<el-dialog :visible.sync="dialogVisible" append-to-body>
						<img width="100%" :src="dialogImageUrl" alt="">
					</el-dialog>
				</el-tab-pane>
				<el-tab-pane label="商品详情" name="detail">
					<my-editor :defaultMsg=defaultMsg :config=config :id=editorId ref="editor"></my-editor>
				</el-tab-pane>
			</el-tabs>
		</el-form>
		<div class="footer">
			<el-button>取 消</el-button>
			<el-button type="primary" @click="addProductSubmit">确 定</el-button>
		</div>
	</div>
</template>

<script>
	import {getCateLsit, getParams} from '../../api/api.js'
	import MyEditor from './MyEditor.vue'
	export default {
		data(){
			return {
				active: 2,
				tabPosition: 'left',
				pform: {
					is_promote: false,
					pcategory: [],
					selectedOptions: [],
					dparam: [],
					dtype: [],
					sparam: [],
					richText: '',
					pictures: [],
					selfDefineAttr: {
						value: 'cat_id',
						label: 'cat_name'
					}
				},
				rules: {},
				nowTab: '',
				dialogVisible: false,
				dialogImageUrl: '',
				token: {authorization: localStorage.getItem('mytoken')},
				baseUrl: 'http://47.96.21.88:8888/api/private/v1/upload',
				defaultMsg: '',
				editorId: 'editorId',
				config: {
					toolbars: [['Source', 'FullScreen', 'simpleupload', 'Undo', 'Redo', 'Bold', 'test']],
					// serverUrl: uploadInfo().url + '/upload',
					serverUrl: 'http://47.96.21.88:8888/ueditor/ue',
					initialFrameWidth: null,
					initialFrameHeight: 350,
					dataType: 'jsonp',
					jsonp: 'hello'
				}
			}
		},
		methods: {
			// 上传图片
			handlePictureCardPreview(){},
			handleSuccess(){},
			handleRemove(){},
			addProductSubmit(){},
			initCate () {
				// 初始化分类列表
				getCateLsit({type: 3}).then(res => {
					if (res.meta.status === 200) {
						this.pform.pcategory = res.data
					}
				})
			},
			toggleTabs(tabPrice){
				if (this.pform.selectedOptions.length !== 3) {
					// 没有选择三级分类，提示并终止运行
					this.$message({
						type: 'error',
						message: '请选择三级分类'
					})
					return
				}
				this.nowTab = tabPrice.name;
				if (this.nowTab === 'params') {
					// 处理动态参数
					this._handleParam(this.pform.selectedOptions[2])
				}  else if (this.nowTab === 'attrs') {
					// 处理静态参数
					this._handleAttrs(this.pform.selectedOptions[2])
				}
			},
			_handleParam (id) {
				// 处理动态参数
				getParams({
					id: id,
					sel: 'many'
				}).then(res => {
					if (res.meta.status === 200) {
						// 获取动态参数
						this.pform.dparam = res.data.map(item => {
							if (item.attr_vals) {
								item.attr_vals = item.attr_vals.split(',')
							} else {
								item.attr_vals = []
							}
							return item
						})
					}
				})
			},
			_handleAttrs (id) {
				// 处理静态属性
				getParams({
					id: id,
					sel: 'only'
				}).then(res => {
					if (res.meta.status === 200) {
						this.pform.sparam = res.data
					}
				})
			}
		},
		mounted(){
			this.initCate()
		},
		components: {
			MyEditor
		}
	}
</script>

<style scoped>
	.title {
		height: 45px;
		line-height: 45px;
		background-color: #D7E2EF;
		padding-left: 20px;
	}
	.el-steps {
		margin: 15px 0px;
		padding-left: 20px;
	}
	.footer {
		margin-top: 10px;
		text-align: center;
	}
</style>
