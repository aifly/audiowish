<template>
	<transition name='index'>
		<div v-if='show' class="lt-full zmiti-index-main-ui " :style='{background:"url("+imgs.createBg+") no-repeat center",backgroundSize:"cover"}'>
			 
		</div>

	</transition>
</template>

<script>
	import './index.css';
	import {imgs} from '../lib/assets.js';
	import {zmitiUtil} from '../lib/util';
	import Point from './point';
	import '../lib/html2canvas.js';
	import Toast from '../toast/toast';
	export default {
		props:['obserable','nickname','pv'],
		name:'zmitiindex',
		data(){
			return{
				imgs,
				errorMsg:"",
				rechooseActive:false,
				sureActive:false,
				createClass:"",
				bg:imgs.index,
				showVideo1:false,
				secretKey: "e9469538b0623783f38c585821459454",
                host: "https://xlive.xinhuaapp.com", //测试域名：https://testxlive.xinhuaapp.com
				showMask:false,
				cacheImg:'',
				viewH:window.innerHeight,
				viewW:window.innerWidth,
				openBtnActive:false,
				indexVideo:window.config.indexvideo,
				showImg:true,
				showPrize:false,
				points:[],
				len:9,
				mobile:'',
				wishes:window.config.wishes,
				show:true,
				showChangeBtn:false,
				start:false,
				showIndex:false,
				showShareMask:false,
				showCanvas:false,
				showNickname:false,
				textIndexArr:[1],
				textInow:0,
				indexArr:[1],
				showQrcode:false,
				imgIndex:1,
				imgDone:false,
				currentStep:0,
				iNow:-1,
				showBitmapBtns:false,
				createImg:'',
				showTeamPage:false,
				successMsg:'',
				longtapChangeActive:false,
				result:{
					img:'',
					wish:''
				},
				showPlayer:true,
			}
		},
		components:{
			Toast
		},
		
		methods:{
			longTapChange(index){
				this.longtapChangeActive = !!index;
			},
			showShare(){
				//this.showShareMask = true;
				this.cacheImg = '';
				this.showQrcode = false;
				this.showNickname = true;
				
				this.restart();
			},
			imgStart(e){
				e.preventDefault(); 
			},

			backtoShare(e){
				
				this.showPrize = false;
			},

			submit(){
				if(!(/^1[34578]\d{9}$/.test(this.mobile))){ 
					this.errorMsg = '手机号格式不正确';
					setTimeout(() => {
						this.errorMsg = '';
					}, 2000);
					return;
				}
				var s = this;
				axios.post(this.host+'/xhs-security-activity/activity/user/saveUser',{
					"secretKey": s.secretKey, // 请求秘钥
					"nm": "gcbn", // 活动某组图片点赞标识 或者活动某组图片浏览量标识 标识由更新接口定义
					uname:s.nickname||"新华社网友",
					unickName:s.nickname||"新华社网友",
					uphone:s.mobile
				}).then((data)=>{
					
					if(typeof data.data === 'string'){
						data = JSON.parse(data.data);
					}
					else {
						data = data.data;
					}
					if(data.rc === 0){
						s.successMsg = '提交成功';

					}else{
						s.errorMsg =  data.msg;
						s.mobile = '';
					}
					setTimeout(() => {
						s.successMsg = '';
						s.errorMsg = '';
						s.showPrize = false;
					}, 2000);
 
				})
			},

			next(){
				this.showMask = false;
				this.showVideo1  = true;
				this.showImg = true;
				//this.indexVideo = window.config.indexvideo1;	
				setTimeout(() => {
					this.$refs['video1'].play();
				}, 1);		
			},
			restart(){
				this.showBitmapBtns = false;
				this.showChangeBtn = true;

				/* this.touchstart();
				setTimeout(() => {
					this.touchend('',true);
				}, 2000); */

			},
			choose(){
				this.bg = imgs['img'+this.indexArr[this.iNow]];
				this.showBitmapBtns = false;
				this.showQrcode = true;
				this.html2img();
			},
			showPrizePage(){//显示抽奖页面
				this.showPrize = true;
			},
			html2img(){
				var s = this;

				var {obserable} = this;

				//document.title = '开始截图....'

				setTimeout(()=>{
					//this.showLoading = true;
					var ref = 'page';
					var dom = this.$refs[ref];
					html2canvas(dom,{
						useCORS: true,
						onrendered: function(canvas) {
					        var src = canvas.toDataURL();
							//s.mergeImg = '';
							//s.createImg = src;
							s.cacheImg = src;

							setTimeout(() => {
								s.createClass = 'active';
							}, 20);
					      },
					      width: dom.clientWidth,
					      height:dom.clientHeight
					})
				},500);
			},
			playVideo(e){
				e.preventDefault();
				if(this.currentStep>0){
					return;
				}

				this.showVideo1 = true;

				var video = this.$refs['video1'];
				this.video = video;
				video.play();


				video.addEventListener('timeupdate',()=>{
					this.showImg = false;
					if(video.currentTime>6&&this.currentStep === 0){
						this.currentStep++;
						video.pause();
						setTimeout(() => {
							this.bg = imgs.index1;
							this.showMask = true;
						}, 2000);
					}
				});

				video.addEventListener('canplaythrough',()=>{
					//this.showPlayer = false;

				});

				video.addEventListener('play',()=>{
					this.showPlayer = false;
					///this.showImg = false;
				 
				})

				video.addEventListener('ended',(e)=>{

					this.bg = imgs.index2;
					
					setTimeout(() => {
						this.showVideo1 = false;//播放完成以后，隐藏视频。
						//this.video.setAttribute('x5-video-player-fullscreen',false);
						//this.video.display = 'none';
					}, 100);
					this.showNickname = true;
					this.showChangeBtn = true;
					this.showImg = true;

					this.obserable.trigger({
						type:'playVoice',
						
					})
					
					
				});
				return false;
			},

			setSize(){
				return;
				var bgratio = 1080/1920;
				if((innerWidth/innerHeight)<=9/16){
					this.$refs['video1'].style.width = parseInt(bgratio*innerHeight) + 'px';
					this.$refs['video1'].style.height = parseInt(innerHeight) + 'px';
				}else{
					this.$refs['video1'].style.width = innerWidth + 'px';
					this.$refs['video1'].style.height = parseInt(innerWidth / bgratio) + 'px';
				}
				//this.$refs['video1'].
			},
			touchstart(e){
				if(e){
					e.preventDefault();
					e.stopPropagation();
				}
				this.obserable.trigger({
					type:'playVoice',
				})
				if(this.iNow <= -1){
					this.iNow = 0;
				}

				this.longtapChangeActive = true;
				clearInterval(this.timer);
				
				this.timer = setInterval(()=>{
					if(this.iNow === 0){
						this.indexArr.sort(()=>{
							return Math.random()>.5;
						});
					}
					this.iNow++;
					this.iNow %= this.len;
					this.bg = imgs['img'+this.indexArr[this.iNow]];
					this.indexArr = this.indexArr.concat([]);
					console.log(this.iNow);
				},200);
				return false;
			},
			randomImg(){
				var time = Math.random()*3000+5000;
				setTimeout(() => {
					clearInterval(this.timer);
					this.showChangeBtn = false;
					this.showBitmapBtns = true;
					this.showCanvas = false;
					this.bg = imgs['gif'+this.indexArr[this.iNow]];
				}, time);
			},
			touchend(e,isNew=false){
				this.longtapChangeActive = false;
				
				

				
			},
			
			end(){
				this.showIndex = true;
			},

			shake(){
				if(!this.showNickname){
					return;
				}
				 if (window.DeviceMotionEvent) {
					   window.addEventListener('devicemotion',deviceMotionHandler,false);
 				}
				var SHAKE_THRESHOLD = 4000;
				var last_update = 0;
				var s = this;
				var x, y, z, last_x = 0, last_y = 0, last_z = 0;
				function deviceMotionHandler(eventData) {
						var acceleration =eventData.accelerationIncludingGravity;
						var curTime = new Date().getTime();
						if ((curTime-last_update)> 10) {
							var diffTime = curTime -last_update;
							last_update = curTime;
							x = acceleration.x;
							y = acceleration.y;
							z = acceleration.z;
							var speed = Math.abs(x +y + z - last_x - last_y - last_z) / diffTime * 10000;
							if (speed > SHAKE_THRESHOLD) {
								if(s.iNow <= -1){
									s.iNow = 0;
								}
								if(s.iNow === 0){
									s.indexArr.sort(()=>{
										return Math.random()>.5;
									});
								}
								s.iNow++;
								s.iNow %= s.len;
								s.bg = imgs['img'+s.indexArr[s.iNow]];
							}
							last_x = x;
							last_y = y;
							last_z = z;
						}
				}
			},
			updatePv(){
				var s = this;
				axios.post(s.host + '/xhs-security-activity/activity/num/updateNum', {
						"secretKey": s.secretKey, // 请求秘钥
						"nm": "gcbn" // 活动某组图片点赞标识 或者活动某组图片浏览量标识 标识由更新接口定义
					}).then(function (data) {
						var dt = data.data;
						if (typeof dt === 'string') {
							dt = JSON.parse(dt);
						}
						console.log(dt);

					});
			}
		},
		mounted(){
			this.setSize();
			this.shake()
			this.updatePv();
			this.indexArr = [];
			
			this.obserable.on('initIndex',()=>{
				this.iNow =  0;
			})
			for(var i =1;i<this.len+1;i++){
				this.indexArr[i-1]= i;
			}
			
		}
	}