<template>
	<view style="position: fixed;left:100%;z-index: -9999;opacity: 0;">
		<canvas :canvas-id="canvasID" :style="{ width: canvasW + 'px', height: canvasH + 'px' }"></canvas>
	</view>
</template>

<script>
	var _this;
	export default {
		name: "printable-coupon",
		props: {
			//canvasID 等同于 canvas-id
			canvasID: {
				Type: String,
				default: 'posterCanvas'
			},
			//画布宽度  (高度为宽度的五分之四自动计算 单位rpx)
			// width: {
			// 	type: String,
			// 	default: '700'
			// },
			//优惠券类型 0代金券 1折扣券 2兑换券
			type: {
				type: Number,
				default: 0
			},
			//是否展示下半部分详情区，默认展示
			showDesc: {
				type: Boolean,
				default: true
			},
			//优惠券左侧标题
			leTitle: {
				//优惠券左侧标题
				type: String,
				default: ''
			},
			//优惠券左侧内容
			leDesc: {
				type: String,
				default: ''
			},
			//优惠券右侧标题
			riTitle: {
				type: String,
				default: ''
			},
			//优惠券开始时间到结束时间 格式 yyyy.mm.dd - yyyy.mm.dd
			riTime: {
				type: String,
				default: ''
			},
			//商家名称
			shop: {
				type: String,
				default: ''
			},
			//商家地址 最多显示28个字 两行，如果超过28个字则显示前27个字并显示三个点
			address: {
				type: String,
				default: ''
			},
			//商家电话
			phone: {
				type: String,
				default: ''
			},
			//商家微信
			wxchat: {
				type: String,
				default: ''
			}
		},
		data() {
			return {
				canvasW: 0,
				canvasH: 0,
				ctx: null,
				width:700
			};
		},
		methods: {
			async onCanvas() {
				uni.showLoading({
					title: "努力加载中.."
				})
				const ctx = uni.createCanvasContext(_this.canvasID, _this);
				ctx.clearRect(0, 0, _this.canvasW, _this.canvasH);
				_this.canvasW = uni.upx2px(_this.width)
				_this.canvasH = uni.upx2px(_this.width * 0.8)
				ctx.setFillStyle('#fff'); //canvas背景颜色
				ctx.fillRect(0, 0, _this.canvasW, _this.canvasH); //canvas画布大小
				//最终显示正常的需要宽为350px 为了让其显示完全 所以需要进行比例缩放
				ctx.scale(_this.canvasW / 350, _this.canvasW / 350)
				//线条颜色
				let lineStroke = '#a3a3a3'
				let beginX = 10 //起始坐标x
				let beginY = 10 //起始坐标y
				let leftW = 100 //左侧宽度
				let rightW = 210 //右侧宽度
				let couponH = 90 //高度
				let radius = 10 //圆的半径
				_this.drawCouponShell(ctx, beginX, beginY, leftW, rightW, couponH, radius, lineStroke)

				//优惠券内部文字(左侧)
				let u = '￥'
				let m = _this.leTitle
				let desc1 = _this.leDesc > 0 ? `满${_this.leDesc}元可用` : '无门槛使用'
				let leftFont = {
					x: beginX + leftW / 2 - radius - ctx.measureText(u).width / 2 - ctx.measureText(m)
						.width / 2,
					y: beginY + 40
				}
				let mX = leftFont.x + ctx.measureText(u).width
				let mY = leftFont.y
				let uX = leftFont.x
				if (_this.type == 1) {
					u = '折'
					mX = leftFont.x
					uX = leftFont.x + ctx.measureText(m).width * 2 + 3
				}
				if (_this.type == 2) {
					u = ''
					desc1 = ''
					mY = leftFont.y + 10
				}
				//优惠券左侧上
				ctx.beginPath();
				_this.textAlign = 'center'
				ctx.setFontSize(uni.upx2px(26));
				ctx.setFillStyle('#E41F19');
				ctx.fillText(u, uX, leftFont.y);
				//大字体  代金券 左小右大 折扣券 左大右小
				ctx.font = 'normal bold 20px sans-serif';
				ctx.fillText(m, mX, mY)
				ctx.closePath();

				//优惠券左侧下
				ctx.beginPath();
				_this.textAlign = 'center'
				ctx.setFontSize(uni.upx2px(24));
				ctx.setFillStyle('#E41F19');
				ctx.fillText(desc1, leftFont.x - ctx.measureText(_this.leDesc).width / 2, leftFont.y + 20)
				ctx.closePath();

				//优惠券内部文字(右侧)
				let title = _this.riTitle
				let desc2 = `领取时间:${_this.riTime}`

				ctx.beginPath();
				//设置字体（粗体 大小 
				ctx.font = 'normal bold 22px sans-serif';
				ctx.setFillStyle('#000');
				ctx.fillText(title, beginX + leftW + radius + 20, leftFont.y - 5)
				ctx.closePath();

				ctx.beginPath();
				ctx.setFontSize(uni.upx2px(23));
				ctx.setFillStyle('#B6B6B6');
				ctx.fillText(desc2, beginX + leftW + radius + 20, leftFont.y + 25)
				ctx.closePath();

				if (_this.showDesc == true) {
					const ws = _this.judgeWidth(_this.width)
					const bottomSet = {
						rimX: beginX,
						rimY: beginY + couponH + beginY,
						rimW: leftW + radius * 2 + rightW,
						rimH: (_this.canvasH - couponH * (_this.canvasW / 350) - beginY * ws * (_this.canvasW /
							350))
					}
					//下侧内容(边框)
					ctx.beginPath();
					ctx.setShadow(1, 1.5, 3, '#a3a3a3')
					ctx.setFillStyle(lineStroke)
					ctx.rect(bottomSet.rimX, bottomSet.rimY, bottomSet.rimW, bottomSet.rimH)
					ctx.stroke();
					ctx.closePath();
					//取消下面的阴影
					ctx.setShadow(0, 0, 0, '#000')

					ctx.setFillStyle('#000');
					ctx.font = 'normal bold 16px sans-serif';
					ctx.fillText('适用门店', bottomSet.rimX + 20, bottomSet.rimY + 27)

					ctx.beginPath();
					ctx.moveTo(bottomSet.rimX + 20, bottomSet.rimY + 40);
					ctx.lineTo(bottomSet.rimW - 10, bottomSet.rimY + 40);
					ctx.strokeStyle = '#F2F1EF';
					ctx.stroke();
					ctx.closePath();

					const icons = {
						shop: '../../static/shop.png',
						address: '../../static/address.png',
						phone: '../../static/phone.png',
						wxchat: '../../static/wxchat.png'
					}
					const imgY = {
						shop: bottomSet.rimY + 55,
						address: bottomSet.rimY + 80,
						phone: bottomSet.rimY + 115,
						wxchat: bottomSet.rimY + 135
					}
					ctx.setFillStyle('#616161');
					ctx.drawImage(icons.shop, bottomSet.rimX + 20, bottomSet.rimY + 52, 15, 15)

					ctx.setFontSize(uni.upx2px(30));
					ctx.fillText(_this.shop, bottomSet.rimX + 50, bottomSet.rimY + 64)
					ctx.drawImage(icons.address, bottomSet.rimX + 20, bottomSet.rimY + 77, 16, 16)
					if (_this.address.length > 14) {
						//地址超过14个字符时则会换行 第一行只能展示14个字符
						ctx.fillText(_this.address.slice(0, 14), bottomSet.rimX + 50, bottomSet.rimY + 90)
						let endStr = _this.address.slice(14)
						if (endStr.length > 14) {
							//如果截取14个后的部分还是大于14则第二行只展示13个字符外加三个点
							ctx.fillText(endStr.slice(0, 13) + '...', bottomSet.rimX + 50, bottomSet.rimY + 108)
						} else {
							//其他时候则会展示完全
							ctx.fillText(endStr, bottomSet.rimX + 50, bottomSet.rimY + 108)
						}
					} else {
						//否则会展示完全
						ctx.fillText(_this.address, bottomSet.rimX + 50, bottomSet.rimY + 90)
					}
					ctx.drawImage(icons.phone, bottomSet.rimX + 20, bottomSet.rimY + 115, 16, 16)
					ctx.fillText(_this.phone, bottomSet.rimX + 50, bottomSet.rimY + 129)
					ctx.drawImage(icons.wxchat, bottomSet.rimX + 20, bottomSet.rimY + 135, 16, 16)
					ctx.fillText(_this.wxchat, bottomSet.rimX + 50, bottomSet.rimY + 149)
				}

				//延迟后渲染至canvas上
				let pic = await _this.setTime(ctx)
				_this.$emit('success', pic);
			},
			//彻底改成同步 防止拿到的图片地址为空
			setTime(ctx) {
				return new Promise((resole, err) => {
					setTimeout(() => {
						ctx.draw(true, async (ret) => {
							let pic = await _this.getNewPic();
							resole(pic)
						});
					}, 600)
				})
			},
			getNewPic() {
				return new Promise((resolve, errs) => {
					uni.canvasToTempFilePath({
						canvasId: _this.canvasID,
						quality: 1,
						complete: (res) => {
							// 在H5平台下，tempFilePath 为 base64
							// 关闭showLoading
							uni.hideLoading();
							//  储存海报地址  也是分享的地址
							resolve(res)
						}
					}, _this);
				})
			},
			//获取图片的临时地址
			getImageInfo(imgSrc) {
				return new Promise((resolve, errs) => {
					setTimeout(() => {
						uni.getImageInfo({
							src: imgSrc,
							success: (image) => {
								resolve(image);
							},
						});
					}, 1000)
				});
			},
			/**
			 * 绘制优惠券外观
			 * @param {Object} ctx canvas的上下文环境
			 * @param {Object} beginX 起始x轴坐标
			 * @param {Object} beginY 起始y轴坐标
			 * @param {Object} leftW 优惠券左侧宽度
			 * @param {Object} rightW 优惠券右侧宽度
			 * @param {Object} couponH 优惠券高度
			 * @param {Object} radius 圆的半径
			 * @param {Object} lineStroke 线条颜色
			 */
			drawCouponShell(ctx, beginX, beginY, leftW, rightW, couponH, radius, lineStroke) {
				//优惠券外壳
				ctx.setShadow(1, 1.5, 3, '#a3a3a3')
				// 左上侧圆角
				ctx.beginPath();
				ctx.arc(beginX + 10, beginY + 10, radius, Math.PI, Math.PI * 1.5);
				ctx.strokeStyle = lineStroke;
				ctx.stroke();
				ctx.closePath();

				//左上横线
				ctx.beginPath();
				ctx.moveTo(beginX + 10, beginY);
				ctx.lineTo(leftW + beginX, beginY);
				ctx.strokeStyle = lineStroke;
				ctx.stroke();
				ctx.closePath();

				//上半圆
				ctx.beginPath();
				ctx.arc(leftW + beginX + radius, beginY, radius, 0, Math.PI);
				ctx.strokeStyle = lineStroke;
				ctx.stroke();
				ctx.closePath();

				//右上横线
				ctx.beginPath();
				ctx.moveTo(beginX + leftW + radius * 2, beginY);
				ctx.lineTo(beginX + leftW + radius * 2 + rightW, beginY);
				ctx.strokeStyle = lineStroke;
				ctx.stroke();
				ctx.closePath();

				//右侧竖线
				ctx.beginPath();
				ctx.moveTo(beginX + leftW + radius * 2 + rightW, beginY);
				ctx.lineTo(beginX + leftW + radius * 2 + rightW, couponH + beginY);
				ctx.strokeStyle = lineStroke;
				ctx.stroke();
				ctx.closePath();

				//右下横线
				ctx.beginPath();
				ctx.moveTo(beginX + leftW + radius * 2, beginY + couponH);
				ctx.lineTo(beginX + leftW + radius * 2 + rightW, beginY + couponH);
				ctx.strokeStyle = lineStroke;
				ctx.stroke();
				ctx.closePath();

				//下半圆
				ctx.beginPath();
				ctx.arc(leftW + beginX + radius, beginY + couponH, radius, 0, Math.PI, true);
				ctx.strokeStyle = lineStroke;
				ctx.stroke();
				ctx.closePath();

				//左下横线
				ctx.beginPath();
				ctx.moveTo(beginX + 10, couponH + beginY);
				ctx.lineTo(leftW + beginX, couponH + beginY);
				ctx.strokeStyle = lineStroke;
				ctx.stroke();
				ctx.closePath();

				//左下侧圆角
				ctx.beginPath();
				ctx.arc(beginX + 10, couponH + beginY - 10, radius, Math.PI * 0.5, Math.PI);
				ctx.stroke();
				ctx.closePath();

				//左侧竖线
				ctx.beginPath();
				ctx.moveTo(beginX, beginY + 10);
				ctx.lineTo(beginX, couponH + beginY - 10);
				ctx.strokeStyle = lineStroke;
				ctx.stroke();
				ctx.closePath();

				//虚线
				ctx.beginPath();
				ctx.moveTo(beginX + leftW + radius, beginY + radius);
				ctx.lineTo(beginX + leftW + radius, beginY + couponH - radius);
				ctx.setLineDash([5, 8]);
				ctx.strokeStyle = lineStroke;
				ctx.stroke();
				ctx.closePath();

				//取消下面的阴影
				ctx.setShadow(0, 0, 0, '#000')
				//取消虚线的使用
				ctx.setLineDash();
			},
			//根据输入的宽_this.width(rpx)判断方框的高度比例
			judgeWidth(width) {
				const dpr = wx.getSystemInfoSync().pixelRatio
				if (width >= 600) {
					if (dpr == 2) {
						return width / 50 - 11
					}
					return width / 50 - 11 + 1
				}
				return (width / 50 - 12) * 2 + 1
			},
			//
		},
		mounted() {
			_this = this;
		},
	}
</script>

<style lang="scss">

</style>
