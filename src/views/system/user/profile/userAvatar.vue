<template>
	<div>
		<div class="user-info-head" @click="editCropper()">
			<img
				v-bind:src="options.img"
				title="点击上传头像"
				class="img-circle img-lg"
			/>
		</div>
		<el-dialog
			:title="title"
			v-model="open"
			width="800px"
			append-to-body
			@opened="modalOpened"
		>
			<el-row>
				<el-col :xs="24" :md="12" :style="{ height: '350px' }">
					<vue-cropper
						ref="cropper"
						:img="options.img"
						:info="true"
						:autoCrop="options.autoCrop"
						:autoCropWidth="options.autoCropWidth"
						:autoCropHeight="options.autoCropHeight"
						:fixedBox="options.fixedBox"
						@realTime="realTime"
						v-if="visible"
					/>
				</el-col>
				<el-col :xs="24" :md="12" :style="{ height: '350px' }">
					<div class="avatar-upload-preview">
						<!-- prettier-ignore -->
						<img :src="options.previews.url" :style="options.previews.img" />
					</div>
				</el-col>
			</el-row>
			<br />
			<el-row>
				<el-col :lg="2" :md="2">
					<el-upload
						action="#"
						:http-request="requestUpload"
						:show-file-list="false"
						:before-upload="beforeUpload"
					>
						<el-button size="small">
							选择
							<i class="upload -right"></i>
						</el-button>
					</el-upload>
				</el-col>
				<el-col :lg="{ span: 1, offset: 2 }" :md="2">
					<el-button
						icon="plus"
						size="small"
						@click="changeScale(1)"
					></el-button>
				</el-col>
				<el-col :lg="{ span: 1, offset: 1 }" :md="2">
					<el-button
						icon="minus"
						size="small"
						@click="changeScale(-1)"
					></el-button>
				</el-col>
				<el-col :lg="{ span: 1, offset: 1 }" :md="2">
					<el-button
						icon="refresh-left"
						size="small"
						@click="rotateLeft()"
					></el-button>
				</el-col>
				<el-col :lg="{ span: 1, offset: 1 }" :md="2">
					<el-button
						icon="refresh-right"
						size="small"
						@click="rotateRight()"
					></el-button>
				</el-col>
				<el-col :lg="{ span: 2, offset: 6 }" :md="2">
					<!-- prettier-ignore -->
					<el-button type="primary" size="small" @click="uploadImg()">提 交</el-button>
				</el-col>
			</el-row>
		</el-dialog>
	</div>
</template>

<script lang="ts" name="UserAvatar" setup>
import useUserStore from "@/store/modules/user";
import { uploadAvatar } from "@/api/system/user";
import { ref, getCurrentInstance, reactive } from "vue";

const baseURL = import.meta.env.VITE_APP_BASE_API;

const { proxy } = getCurrentInstance() as any;
// 是否显示弹出层
const open = ref<boolean>(false);
// 是否显示cropper
const visible = ref<boolean>(false);
// 弹出层标题
const title = ref<string>("修改头像");
const options = reactive<any>({
	img: useUserStore().avatar, //裁剪图片的地址
	autoCrop: true, // 是否默认生成截图框
	autoCropWidth: 200, // 默认生成截图框宽度
	autoCropHeight: 200, // 默认生成截图框高度
	fixedBox: true, // 固定截图框大小 不允许改变
	previews: {
		url: "",
		img: "",
	},
});

// 编辑头像
const editCropper = () => {
	open.value = true;
};
// 打开弹出层结束时的回调
const modalOpened = () => {
	visible.value = true;
};
// 覆盖默认的上传行为
const requestUpload: any = () => {};
// 向左旋转
const rotateLeft = () => {
	proxy.$refs.cropper.rotateLeft();
};
// 向右旋转
const rotateRight = () => {
	proxy.$refs.cropper.rotateRight();
};
// 图片缩放
const changeScale = (num: any) => {
	num = num || 1;
	proxy.$refs.cropper.changeScale(num);
};
// 上传预处理
const beforeUpload = (file: any) => {
	if (file.type.indexOf("image/") == -1) {
		// prettier-ignore
		proxy.$modal.msgError("文件格式错误，请上传图片类型,如：JPG，PNG后缀的文件。");
	} else {
		const reader = new FileReader();
		reader.readAsDataURL(file);
		reader.onload = () => {
			options.img = reader.result;
		};
	}
};
// 上传图片
const uploadImg = () => {
	proxy.$refs.cropper.getCropBlob((data: any) => {
		let formData = new FormData();
		formData.append("avatarfile", data);
		uploadAvatar(formData).then((response: any) => {
			if (response.code === 200) {
				open.value = false;
				options.img = baseURL + response.data;
				useUserStore().avatar = options.img;
				proxy.$modal.msgSuccess("修改成功");
				visible.value = false;
			}
		});
	});
};
// 实时预览
const realTime = (data: any) => {
	options.previews = data;
};
</script>
<style scoped lang="scss">
.user-info-head {
	position: relative;
	display: inline-block;
	height: 120px;
}

.user-info-head:hover:after {
	content: "+";
	position: absolute;
	left: 0;
	right: 0;
	top: 0;
	bottom: 0;
	color: #eee;
	background: rgba(0, 0, 0, 0.5);
	font-size: 24px;
	font-style: normal;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
	cursor: pointer;
	line-height: 110px;
	border-radius: 50%;
}
</style>
