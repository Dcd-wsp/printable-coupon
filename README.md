# printable-coupon

一个适用于uniapp微信小程序端的生成优惠券分享图片（5:4）的组件

可微信扫码该二维码进行测试：         <img src="https://vkceyugu.cdn.bspapp.com/VKCEYUGU-8e16d550-f6e0-406f-9b1e-0d2439dfc4c7/3a017b83-7644-4139-b21a-62cbefbf8e9e.png" style="zoom: 33%;" />

## 安装

有三种安装方式

1. 使用[npm](https://www.npmjs.com/)

   ```
   npm i printable-coupon
   ```

2. 如果您使用[HBuilderX](https://www.dcloud.io/hbuilderx.html)创建的`uniapp`项目，点击跳转至插件市场查看该[项目](https://ext.dcloud.net.cn/plugin?id=5313)

3. 手动下载或者git clone 本项目，
   将其中的[printable-coupon](https://github.com/Dcd-wsp/printable-coupon/tree/main/printable-coupon)文件夹及内容放入项目中的`components`文件夹内，
   用时直接`import`即可


在`main.js`中导入本组件

```js
import printableCoupon from "printable-coupon";

Vue.use(printableCoupon);
```

或者直接在用到该组件的页面`import`导入

在需要的地方，通过`ref`的方式调用`onCanvas()`即可在事件`success`中拿到图片的地址

**ps：**如控制台报错 `getImageInfo:xxxxxxxxx...` 请先检查详情区的四个图标有无配置download域名白名单

​	为保证速度，用时最好下载下来存放于自己的图片服务器中

## 属性说明：

|  属性名  | 是否必须 |  类型   |            默认值            |                             说明                             |
| :------: | :------: | :-----: | :--------------------------: | :----------------------------------------------------------: |
| canvasID |    否    | String  |         posterCanvas         |          等同于 canvas-id，不报错的情况下不建议修改          |
|   type   |    否    | Number  |              0               |              优惠券类型 0代金券 1折扣券 2兑换券              |
| leTitle  |    否    | String  |                              | 优惠券内部左侧显示的金额或折扣。当type为2时固定显示为兑换券  |
|  leDesc  |    否    | Number  |                              | 优惠券内部左侧满减金额。当金额<=0时，显示【无门槛使用】。当type为2时不显示 |
| riTitle  |    否    | String  |                              |                      优惠券内部右侧标题                      |
|  riTime  |    否    | String  |                              |                  优惠券内部右侧领取时间限制                  |
| showDesc |    否    | Boolean |             true             |               是否展示下半部分详情区，默认展示               |
|   shop   |    否    | String  |                              |             商家名称。在showDesc为true时才会展示             |
| address  |    否    | String  |                              | 商家地址（最多显示28个字，共两行，如果超过28个字则显示前27个字并显示三个点）。在showDesc为true时才会展示 |
|  phone   |    否    | String  |                              |             商家电话。在showDesc为true时才会展示             |
|  wxchat  |    否    | String  |                              |             商家微信。在showDesc为true时才会展示             |
|  icons   |    否    | Object  | shop，address，phone，wxchat | 详情区用到的icons。在showDesc为true时才会展示。强烈建议更改为自己图片服务器的网址 |

​    详情区用到的icons ：   <img src="https://vkceyugu.cdn.bspapp.com/VKCEYUGU-8e16d550-f6e0-406f-9b1e-0d2439dfc4c7/e47003dd-8ea2-40c7-b48f-aede0b9af3df.png" style="zoom:25%;" /><img src="https://vkceyugu.cdn.bspapp.com/VKCEYUGU-8e16d550-f6e0-406f-9b1e-0d2439dfc4c7/4a1dd675-6368-4a9b-86e1-d54b77e3768f.png" style="zoom:25%;" /><img src="https://vkceyugu.cdn.bspapp.com/VKCEYUGU-8e16d550-f6e0-406f-9b1e-0d2439dfc4c7/f7fc3cd5-9d20-48d1-ad94-e9d9583fd2f6.png" style="zoom:25%;" /><img src="https://vkceyugu.cdn.bspapp.com/VKCEYUGU-8e16d550-f6e0-406f-9b1e-0d2439dfc4c7/95d64721-5359-4211-b3cb-168b8a8062f4.png" style="zoom:25%;" />

## 事件说明：

| 事件称名 |                     说明                     |
| :------: | :------------------------------------------: |
| success  | success(e){  console.log('临时图片地址:',e } |

例子：

```vue
<template>
	<view>
		<printable-coupon ref="poster"
			:type="current" :leTitle="coupon.num" 
			:leDesc="coupon.price" :riTitle="coupon.title" 
			:riTime="coupon.time" :showDesc="shopShow==0?true:false"
			:shop="desc.shopName" :address="desc.address"
			:phone="desc.phone" :wxchat="desc.wxchat"
			@success="shareSuccess"></printable-coupon>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				imgSrc:'',
				shopShow:0,
				coupon:{
					num:'',
					price:'',
					title:'',
					time:'',
				},
				desc:{
					shopName:'',
					address:'',
					phone:'',
					wxchat:''
				}
			}
		},
		async onShareAppMessage(res) {
            //调用组件中的方法
			await  this.$refs.poster.onCanvas()
			return {
				//标题
				title: '测试',
				//图片 即通过该组件生成的图片地址
				imageUrl: this.imgSrc,
			};
		},
		methods: {
			shareSuccess(e){
				console.log('imgSrc:',e)
				this.imgSrc = e
			},
		}
	}
</script>

```

# License

[MIT](https://github.com/eggjs/egg/blob/master/LICENSE)