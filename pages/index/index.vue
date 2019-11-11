<template>
	<view class="container">
		<view class="contents">
			<canvas class="page-content" canvas-id="shareCanvas" style="width:100%;" :style="{ height: height + 'px' }"></canvas>
		</view>
		<view class="page-bottom flex text-center">
			<button class="cu-btn block bg-orange" @tap="selectImage">选择图片</button>
			<button class="cu-btn block bg-orange" @tap="saveImage">保存图库</button>
		</view>
	</view>
</template>

<script>
import tools from '../util/promise.js';

export default {
	components: {  },
	data() {
		return {
			ifShow: false,
			val: '', // 要生成的二维码值
			size: 320, // 二维码大小
			unit: 'upx', // 单位
			background: '#b4e9e2', // 背景色
			foreground: '#309286', // 前景色
			pdground: '#32dbc6', // 角标色
			icon: '', // 二维码图标
			iconsize: 40, // 二维码图标大小
			lv: 3, // 二维码容错级别 ， 一般不用设置，默认就行
			onval: false, // val值变化时自动重新生成二维码
			loadMake: true, // 组件加载完成后自动生成二维码
			src: '', // 二维码生成后的图片地址或base64
			token: 0,
			id: 0,
			height: 500,
			backgroundImage:'https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1573467551191&di=1ee6a75f8bc42dd26bec229a1e572f4c&imgtype=0&src=http%3A%2F%2Fpic.51yuansu.com%2Fbackgd%2Fcover%2F00%2F35%2F24%2F5bd723e3dc2b7.jpg%2521%2Ffw%2F780%2Fquality%2F90%2Funsharp%2Ftrue%2Fcompress%2Ftrue'
		};
	},
	onLoad(options) {
		if(options.img){
			console.log(options.img);
			this.backgroundImage=decodeURIComponent(options.img);;
		}
		
	},
	onShow() {
		this.onInit()
	},
	methods: {
		onInit:function(){
			//微信服务器加载图片
			//请求url  需要从服务器获取base64字节码
			const url ="https://zs.homeeyes.cn/api/v1/getShareImge";
			//加载页面参数
			let  scene = 'id=' + this.id + '&token=' + this.token + '&type=1';
			let params={
			  page: 'pages/index/index',
			  scene: scene
			}
			uni.request({
			    url: url, 
			    data: params,
			    method: "POST",
			    header: {
			    	"Content-Type": "application/x-www-form-urlencoded"
			    },
			    dataType: 'arraybuffer',//请求字节
			    success: (res) => {
					//获取base64 字符串
					var data=res.data;
					//拼装base64图片
					var img='data:image/png;base64,'+data;
					//调起绘制器
					this.initImage(img);
			    }
			});
		},
		initImage: function(img) {
			//加载图片到画布
			let that = this;
			const wxGetImageInfo = tools.promisify(uni.getImageInfo);
			Promise.all([
				wxGetImageInfo({
					src: that.backgroundImage
				}),img
			]).then(res => {
				uni.getSystemInfo({
					success: function(e) {
						console.log(e);
						//获取屏幕宽高设置画布宽高
						let width = e.windowWidth;
						let height= e.screenHeight;
						that.height=height;
						let topheight = 320;//图片距离上边距离
						const ctx = uni.createCanvasContext('shareCanvas');
						// 底图
						ctx.drawImage(res[0].path, 0, 0, width, height);
						//绘制序列从上到下， 通过距离上边高度来排列显示
						// 文字内容
						ctx.setTextAlign('center'); // 文字居中
						ctx.setFillStyle('#F0AD4E'); // 文字颜色：黑色
						ctx.setFontSize(22); // 文字字号：22px
						ctx.fillText('双十一活动', width / 2, topheight-30);
						ctx.setFillStyle('#FFFFFF'); 
							ctx.setFontSize(20); 
						ctx.fillText('扫码领取更多奖', width / 2, topheight);
						// 小程序码
						const qrImgSize = 140;
						ctx.drawImage(res[1], (width - qrImgSize) / 2, topheight+15 , qrImgSize, qrImgSize);
						ctx.stroke();
						ctx.draw();
					}
				});
			});
		},
		saveImage: function() {
			//保存图片到相册
			const wxCanvasToTempFilePath = tools.promisify(uni.canvasToTempFilePath);
			const wxSaveImageToPhotosAlbum = tools.promisify(uni.saveImageToPhotosAlbum);
			wxCanvasToTempFilePath(
				{
					canvasId: 'shareCanvas'
				},
				this
			).then(res => {
					return wxSaveImageToPhotosAlbum({
						filePath: res.tempFilePath
					});
				})
				.then(res => {
					wx.showToast({
						title: '已保存到相册'
					});
				});
		},selectImage(){
			uni.navigateTo({
				url:'cropimage'
			})
		}
	}
};
</script>

<style scoped>
.page-bottom button {
	margin-left: 10upx;
	margin-right: 10upx;
}
.page-bottom {
	padding: 20upx;
	position: fixed;
	bottom: 0;
	left: 0;
	right: 0;
}
.cu-btn{
	width: 100%;
	margin-top: 10upx;
}
</style>
