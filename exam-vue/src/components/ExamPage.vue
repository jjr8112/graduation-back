<template>

  <el-container v-if="show">
    <el-header style="margin-top: 60px">
      <el-row>
        <el-col :span="18" :offset="3" style="border-bottom: 1px solid #f5f5f5">
          <span class="startExam">开始练习</span>
          <span class="examTitle">距离练习结束还有：</span>
          <span style="color: red;font-size: 18px;">{{ duration | timeFormat }}</span>
          <el-button type="warning" round
                     style="background-color: #ffd550;float: right;color: black;font-weight: 800"
                     @click="uploadExamToAdmin">提交试卷
          </el-button>
        </el-col>
      </el-row>
    </el-header>

    <el-main>
      <el-row>
        <el-col :span="18" :offset="3">
          <el-col :span="16">
            <el-card style="min-height: 500px">
              <!--题目信息-->
              <div>
                <i class="num">{{ curIndex+1 }}</i>
                <span v-if="questionInfo[curIndex].questionType === 1">【单选题】</span>
                <span v-else-if="questionInfo[curIndex].questionType === 2">【多选题】</span>
                <span v-else-if="questionInfo[curIndex].questionType === 3">【判断题】</span>
                <span v-else>【简答题】</span>
                <span>{{ questionInfo[curIndex].questionContent}}:</span>
              </div>
              <!--题目中的配图-->
              <img v-for="(url, index) in questionInfo[curIndex].images" :key="index" :src="url" title="点击查看大图" alt="题目图片"
                   style="width: 100px;height: 100px;cursor: pointer" @click="showBigImg(url)">

              <!--单选 和 判断 的答案列表-->
              <div style="margin-top: 25px"
                   v-show="questionInfo[curIndex].questionType === 1 || questionInfo[curIndex].questionType === 3">
                <div class="el-radio-group">
                  <label v-for="(item,index) in questionInfo[curIndex].answer"
                         @click="checkSingleAnswer(index)"
                         :class="index === userAnswer[curIndex] ? 'active' : ''">
                    <span>{{ optionName[index] + '、' + item.answer}}</span>
                    <img style="position: absolute;left:100%;top:50%;transform: translateY(-50%);
                  width: 40px;height: 40px;float: right;cursor: pointer;" title="点击查看大图"
                         v-if="item.images !== null"
                         v-for="i2 in item.images" :src="i2" alt="" @mouseover="showBigImg(i2)">
                  </label>
                </div>
              </div>

              <!--多选的答案列表-->
              <div style="margin-top: 25px" v-show="questionInfo[curIndex].questionType === 2">
                <div class="el-radio-group">
                  <label v-for="(item,index) in questionInfo[curIndex].answer"
                         @click="selectedMultipleAnswer(index)"
                         :class="(userAnswer[curIndex]+'').indexOf(index+'') !== -1? 'active' : ''">
                    <span>{{ optionName[index] + '、' + item.answer}}</span>
                    <img style="position: absolute;left:100%;top:50%;transform: translateY(-50%);
                  width: 40px;height: 40px;float: right;cursor: pointer;" title="点击查看大图"
                         v-if="item.images !== null"
                         v-for="i2 in item.images" :src="i2" alt="" @mouseover="showBigImg(i2)">
                  </label>
                </div>
              </div>

              <!--简答题的答案-->
              <div style="margin-top: 25px" v-show="questionInfo[curIndex].questionType === 4">
                <el-input
                  type="textarea"
                  :rows="8"
                  placeholder="请输入答案"
                  v-model="userAnswer[curIndex]">
                </el-input>
              </div>

              <!--上一题 下一题-->
              <div style="margin-top: 25px">
                <el-button type="primary" icon="el-icon-back" :disabled="curIndex<1" @click="curIndex--">上一题</el-button>
                <el-button type="primary" icon="el-icon-right" :disabled="curIndex>=questionInfo.length-1"
                           @click="curIndex++">下一题
                </el-button>
              </div>

            </el-card>
          </el-col>

          <el-col :span="7" :offset="1">
            <!--答题卡卡片-->
            <el-card>
              <div>
                <p style="font-size: 18px;">答题卡</p>
                <div style="margin-top: 25px">
                  <span
                    style="background-color: rgb(238,238,238);padding: 5px 10px 5px 10px;margin-left: 15px">未作答</span>
                  <span style="background-color: rgb(87,148,247);color: white;
                padding: 5px 10px 5px 10px;margin-left: 15px">已作答</span>
                </div>
              </div>

              <!--单选的答题卡-->
              <div style="margin-top: 25px">
                <p style="font-size: 18px;">单选题</p>
                <el-button style="margin-top: 10px;margin-left: 15px" size="mini"
                           v-show="questionInfo[item-1].questionType === 1"
                           :class="questionInfo[item-1].questionType === 1 && userAnswer[item-1] !== undefined ?
                            'done' : userAnswer[item-1] === undefined ? curIndex === (item-1) ? 'orange' : 'noAnswer' : 'noAnswer'"
                           v-for="item in questionInfo.length" :key="item" @click="curIndex = item-1">{{ item }}
                </el-button>
              </div>

              <!--多选的答题卡-->
              <div style="margin-top: 25px">
                <p style="font-size: 18px;">多选题</p>
                <el-button style="margin-top: 10px;margin-left: 15px" size="mini"
                           v-show="questionInfo[item-1].questionType === 2"
                           :class="questionInfo[item-1].questionType === 2 && userAnswer[item-1] !== undefined ?
                            'done' : userAnswer[item-1] === undefined ? curIndex === (item-1) ? 'orange' : 'noAnswer' : 'noAnswer'"
                           v-for="item in questionInfo.length" :key="item" @click="curIndex = item-1">{{ item }}
                </el-button>
              </div>

              <!--判断的答题卡-->
              <div style="margin-top: 25px">
                <p style="font-size: 18px;">判断题</p>
                <el-button style="margin-top: 10px;margin-left: 15px" size="mini"
                           v-show="questionInfo[item-1].questionType === 3"
                           :class="questionInfo[item-1].questionType === 3 && userAnswer[item-1] !== undefined ?
                            'done' : userAnswer[item-1] === undefined ? curIndex === (item-1) ? 'orange' : 'noAnswer' : 'noAnswer'"
                           v-for="item in questionInfo.length" :key="item" @click="curIndex = item-1">{{ item }}
                </el-button>
              </div>

              <!--简答的答题卡-->
              <div style="margin-top: 25px">
                <p style="font-size: 18px;">简答题</p>
                <el-button style="margin-top: 10px;margin-left: 15px" size="mini"
                           v-show="questionInfo[item-1].questionType === 4"
                           :class="questionInfo[item-1].questionType === 4 && userAnswer[item-1] !== undefined ?
                            'done' : userAnswer[item-1] === undefined ? curIndex === (item-1) ? 'orange' : 'noAnswer' : 'noAnswer'"
                           v-for="item in questionInfo.length" :key="item" @click="curIndex = item-1">{{ item }}
                </el-button>
              </div>
            </el-card>
          </el-col>

        </el-col>
      </el-row>
      <!-- <video id="video" muted="muted" style="float:right;position: fixed;top: 80%;left: 85%" width="200px"
             height="200px" autoplay="autoplay"></video> -->
      <canvas id="canvas" hidden width="200px" height="200px"></canvas>
    </el-main>
    <!--图片回显-->
    <el-dialog :visible.sync="bigImgDialog" @close="bigImgDialog = false">
      <img style="width: 100%" :src="bigImgUrl">
    </el-dialog>
  </el-container>

</template>

<script>

  export default {
    name: 'ExamPage',
    data () {
      return {
        //当前练习的信息
        examInfo: {},
        //当前的练习题目
        questionInfo: [
          {
            questionType: ''
          }
        ],
        //当前题目的索引值
        curIndex: 0,
        //控制大图的对话框
        bigImgDialog: false,
        //当前要展示的大图的url
        bigImgUrl: '',
        //用户选择的答案
        userAnswer: [],
        //页面数据加载
        loading: {},
        //页面绘制是否开始
        show: false,
        //答案的选项名abcd数据
        optionName: ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z'],
        //练习总时长
        duration: 0,
        //摄像头对象
        mediaStreamTrack: null,
        //诚信照片的url
        takePhotoUrl: [],
        //摄像头是否开启
        cameraOn: false,
      }
    },
    created () {
      this.getExamInfo()
      //页面数据加载的等待状态栏
      this.loading = this.$Loading.service({
        body: true,
        lock: true,
        text: '数据拼命加载中,(*╹▽╹*)',
        spinner: 'el-icon-loading',
      })
      //开启摄像头
      // window.onload = () => {
      //   setTimeout(() => {
      //     this.getCamera()
      //   }, 1000)

      //   //生成3次时间点截图
      //   let times = []
      //   for (let i = 0; i < 2; i++) {
      //     times.push(Math.ceil(Math.random() * this.duration * 1000))
      //   }
      //   times.push(10000)
      //   //一次练习最多3次随机的诚信截图
      //   times.forEach(item => {
      //     window.setTimeout(() => {
      //       this.takePhoto()
      //     }, item)
      //   })
      // }
    },
    mounted () {
      //关闭浏览器窗口的时候移除 localstorage的时长
      var userAgent = navigator.userAgent //取得浏览器的userAgent字符串
      var isOpera = userAgent.indexOf('Opera') > -1 //判断是否Opera浏览器
      var isIE = userAgent.indexOf('compatible') > -1 && userAgent.indexOf('MSIE') > -1 && !isOpera //判断是否IE浏览器
      var isIE11 = userAgent.indexOf('rv:11.0') > -1 //判断是否是IE11浏览器
      var isEdge = userAgent.indexOf('Edge') > -1 && !isIE //判断是否IE的Edge浏览器
      if (!isIE && !isEdge && !isIE11) {//兼容chrome和firefox
        var _beforeUnload_time = 0, _gap_time = 0
        var is_fireFox = navigator.userAgent.indexOf('Firefox') > -1//是否是火狐浏览器
        window.onunload = function () {
          _gap_time = new Date().getTime() - _beforeUnload_time
          if (_gap_time <= 5) {
            localStorage.removeItem('examDuration' + this.examInfo.examId)
          } else {//谷歌浏览器刷新
          }
        }
        window.onbeforeunload = function () {
          _beforeUnload_time = new Date().getTime()
          if (is_fireFox) {//火狐关闭执行

          } else {//火狐浏览器刷新
          }
        }
      }
    },
    methods: {
      //查询当前练习的信息
      getExamInfo () {
        this.$http.get(this.API.getExamInfoById, { params: this.$route.params }).then((resp) => {
          if (resp.data.code === 200) {
            this.examInfo = resp.data.data
            //设置定时(秒)
            if (localStorage.getItem('examDuration' + this.examInfo.examId) === '0') 
            localStorage.removeItem('examDuration' + this.examInfo.examId)
            this.duration = localStorage.getItem('examDuration' + this.examInfo.examId) || resp.data.data.examDuration * 60
            console.log(localStorage.getItem('examDuration' + this.examInfo.examId))
            console.log(this.duration)
            //练习剩余时间定时器
            this.timer = window.setInterval(() => {
              if (this.duration > 0) this.duration--
            }, 1000)
            this.getQuestionInfo(resp.data.data.questionIds.split(','))
          }
        })
      },
      //查询练习的题目信息
      async getQuestionInfo (ids) {
        await ids.forEach((item, index) => {
          this.$http.get(this.API.getQuestionById + '/' + item).then((resp) => {
            if (index === 0) this.questionInfo = []
            if (resp.data.code === 200) {
              this.questionInfo.push(resp.data.data)
              //重置问题的顺序 单选 多选 判断 简答
              this.questionInfo = this.questionInfo.sort(function (a, b) {
                return a.questionType - b.questionType
              })
            }
          })
        })
        this.loading.close()
        this.show = true
      },
      //点击展示高清大图
      showBigImg (url) {
        this.bigImgUrl = url
        this.bigImgDialog = true
      },
      //检验单选题的用户选择的答案
      checkSingleAnswer (index) {
        this.$set(this.userAnswer, this.curIndex, index)
      },
      //多选题用户的答案选中
      selectedMultipleAnswer (index) {
        if (this.userAnswer[this.curIndex] === undefined) {//当前是多选的第一个答案
          this.$set(this.userAnswer, this.curIndex, index)
        } else if (String(this.userAnswer[this.curIndex]).split(',').includes(index + '')) {//取消选中
          let newArr = []
          String(this.userAnswer[this.curIndex]).split(',').forEach(item => {
            if (item !== '' + index) newArr.push(item)
          })
          if (newArr.length === 0) {
            this.$set(this.userAnswer, this.curIndex, undefined)
          } else {
            this.$set(this.userAnswer, this.curIndex, newArr.join(','))
            //答案格式化顺序DBAC -> ABCD
            this.userAnswer[this.curIndex] = String(this.userAnswer[this.curIndex]).split(',').sort(function (a, b) {
              return a - b
            }).join(',')
          }
        } else if (!((this.userAnswer[this.curIndex] + '').split(',').includes(index + ''))) {//第n个答案
          this.$set(this.userAnswer, this.curIndex, this.userAnswer[this.curIndex] += ',' + index)
          //答案格式化顺序DBAC -> ABCD
          this.userAnswer[this.curIndex] = String(this.userAnswer[this.curIndex]).split(',').sort(function (a, b) {
            return a - b
          }).join(',')
        }
      },
      //调用摄像头
      getCamera () {
        let constraints = {
          video: {
            width: 200,
            height: 200
          },
          audio: false
        }
        //获得video摄像头
        let video = document.getElementById('video')
        let promise = navigator.mediaDevices.getUserMedia(constraints)
        promise.then((mediaStream) => {
          this.mediaStreamTrack = typeof mediaStream.stop === 'function' ? mediaStream : mediaStream.getTracks()[1]
          video.srcObject = mediaStream
          video.play()
          this.cameraOn = true
        }).catch((back) => {
          this.$message({
            duration: 1500,
            message: '请开启摄像头权限o(╥﹏╥)o!',
            type: 'error'
          })
        })
      },
      //拍照
      takePhoto () {
        if (this.cameraOn) {//摄像头是否开启 开启了才执行上传信用图片
          //获得Canvas对象
          let video = document.getElementById('video')
          let canvas = document.getElementById('canvas')
          let ctx = canvas.getContext('2d')
          ctx.drawImage(video, 0, 0, 200, 200)
          // toDataURL  ---  可传入'image/png'---默认, 'image/jpeg'
          let img = document.getElementById('canvas').toDataURL()

          //构造post的form表单
          let formData = new FormData()
          //convertBase64UrlToBlob函数是将base64编码转换为Blob
          formData.append('file', this.base64ToFile(img, 'examTakePhoto.png'))
          //上传阿里云OSS
          this.$http.post(this.API.uploadQuestionImage, formData).then((resp) => {
            if (resp.data.code === 200) this.takePhotoUrl.push(resp.data.data)
          })
        }
      },
      //关闭摄像头
      closeCamera () {
        let stream = document.getElementById('video').srcObject
        let tracks = stream.getTracks()
        tracks.forEach(function (track) {
          track.stop()
        })
        document.getElementById('video').srcObject = null
      },
      //将摄像头截图的base64串转化为file提交后台
      base64ToFile (urlData, fileName) {
        let arr = urlData.split(',')
        let mime = arr[0].match(/:(.*?);/)[1]
        let bytes = atob(arr[1]) // 解码base64
        let n = bytes.length
        let ia = new Uint8Array(n)
        while (n--) {
          ia[n] = bytes.charCodeAt(n)
        }
        return new File([ia], fileName, { type: mime })
      },
      //上传用户练习信息进入后台
      async uploadExamToAdmin () {
        if (this.cameraOn) await this.takePhoto()//结束的时候拍照上传一张
        // 正则
        var reg = new RegExp('-', 'g')
        // 去掉用户输入的非法分割符号(-),保证后端接受数据处理不报错
        this.userAnswer.forEach((item, index) => {
          if (this.questionInfo[index].questionType === 4) {//简答题答案处理
            this.userAnswer[index] = item.replace(reg, ' ')
          }
        })

        // 标记题目是否全部做完
        let flag = true
        for (let i = 0; i < this.userAnswer.length; i++) {// 检测用户是否题目全部做完
          if (this.userAnswer[i] === undefined) {
            flag = false
          }
        }
        // 如果用户所有答案的数组长度小于题目长度,这个时候也要将标志位置为false
        if (this.userAnswer.length < this.questionInfo.length) {
          flag = false
        }
        //题目未做完
        if (!flag) {
          // if (this.userAnswer.some((item) => item === undefined)) {
          this.$confirm('当前试题暂未做完, 是否继续提交o(╥﹏╥)o ?', 'Tips', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }).then(() => {
            let data = {}
            data.questionIds = []
            data.userAnswers = this.userAnswer.join('-')
            this.questionInfo.forEach((item, index) => {
              data.questionIds.push(item.questionId)
              //当前数据不完整,用户回答不完整(我们自动补充空答案,防止业务出错)
              if (index > (this.userAnswer.length - 1)) {
                data.userAnswers += '- '
              }
            })
            //如果所有题目全部未答
            if (this.userAnswer.length === 0) {
              this.questionInfo.forEach(item => {
                data.userAnswers += ' -'
              })
              data.userAnswers.split(0, data.userAnswers.length - 1)
            }
            data.examId = parseInt(this.$route.params.examId)
            data.questionIds = data.questionIds.join(',')
            data.creditImgUrl = this.takePhotoUrl.join(',')
            this.$http.post(this.API.addExamRecord, data).then((resp) => {
              if (resp.data.code === 200) {
                this.$notify({
                  title: 'Tips',
                  message: '练习结束 *^▽^*',
                  type: 'success',
                  duration: 2000
                })
                this.$router.push('/examResult/' + resp.data.data)
              }
            })
          }).catch(() => {
            this.$notify({
              title: 'Tips',
              message: '继续加油! *^▽^*',
              type: 'success',
              duration: 2000
            })
          })
        } else {//当前题目做完了
          if (this.cameraOn) {
            //结束的时候拍照上传一张
            this.takePhoto()
            this.closeCamera()
          }
          let data = {}
          data.questionIds = []
          data.userAnswers = this.userAnswer.join('-')
          data.examId = parseInt(this.$route.params.examId)
          data.creditImgUrl = this.takePhotoUrl.join(',')
          this.questionInfo.forEach((item, index) => {
            data.questionIds.push(item.questionId)
          })
          data.questionIds = data.questionIds.join(',')
          this.$http.post(this.API.addExamRecord, data).then((resp) => {
            if (resp.data.code === 200) {
              this.$notify({
                title: 'Tips',
                message: '练习结束 *^▽^*',
                type: 'success',
                duration: 2000
              })
              this.$router.push('/examResult/' + resp.data.data)
            }
          })
        }
      }
    },
    watch: {
      //监控练习的剩余时间
      duration (newVal) {
        localStorage.setItem('examDuration' + this.examInfo.examId, newVal)
        //摄像头数据
        // let constraints = {
        //   video: {
        //     width: 200,
        //     height: 200
        //   },
        //   audio: false
        // }
        // //通过调用摄像头判断用户是否中途关闭摄像头
        // let promise = navigator.mediaDevices.getUserMedia(constraints)
        // promise.catch((back) => {
        //   this.cameraOn = false
        // })
        // if (!this.cameraOn) {//如果摄像头未开启,就再次调用开启
        //   this.getCamera()
        // }
        //练习时间结束了提交试卷
        if (newVal < 1) {
          // if (this.cameraOn) {
          //   //结束的时候拍照上传一张
          //   this.takePhoto()
          //   this.closeCamera()
          // }
          let data = {}
          data.questionIds = []
          data.userAnswers = this.userAnswer.join('-')
          this.questionInfo.forEach((item, index) => {
            data.questionIds.push(item.questionId)
            //当前数据不完整,用户回答不完整(我们自动补充空答案,防止业务出错)
            if (index > this.userAnswer.length) {
              data.userAnswers += ' -'
            }
          })
          //如果所有题目全部未答
          if (data.userAnswers === '') {
            this.questionInfo.forEach(item => {
              data.userAnswers += ' -'
            })
            data.userAnswers.split(0, data.userAnswers.length - 1)
          }
          data.examId = parseInt(this.$route.params.examId)

          data.questionIds = data.questionIds.join(',')
          data.creditImgUrl = this.takePhotoUrl.join(',')
          this.$http.post(this.API.addExamRecord, data).then((resp) => {
            if (resp.data.code === 200) {
              this.$notify({
                title: 'Tips',
                message: '练习时间结束,已为您自动提交 *^▽^*',
                type: 'success',
                duration: 2000
              })
              this.$router.push('/examResult/' + resp.data.data)
            }
          })
        }
      },
    }
  }
</script>

<style scoped lang="scss">
  * {
    font-weight: 800;
  }

  .el-container {
    width: 100%;
    height: 100%;
  }

  .startExam {
    color: #160f58;
    border-bottom: 4px solid #ffd550;
    font-size: 18px;
    font-weight: 700;
    padding-bottom: 10px
  }

  .examTitle {
    font-size: 18px;
    color: #cbcacf;
    margin-left: 20px;
    font-weight: 700;
  }

  .el-radio-group label {
    display: block;
    width: 400px;
    padding: 48px 20px 10px 20px;
    border-radius: 4px;
    border: 1px solid #dcdfe6;
    margin-bottom: 10px;
    cursor: pointer;
    position: relative;

    span {
      position: absolute;
      top: 50%;
      transform: translateY(-50%);
      font-size: 16px;
    }
  }

  .el-radio-group label:hover {
    background-color: rgb(245, 247, 250);
  }

  /*当前选中的答案*/
  .active {
    border: 1px solid #1f90ff !important;
    opacity: .5;
  }

  /*做过的题目的高亮颜色*/
  .done {
    background-color: rgb(87, 148, 247);
  }

  /*未做题目的高亮颜色*/
  .noAnswer {
    background-color: rgb(238, 238, 238);
  }

  /*当前在做的题目高亮的颜色*/
  .orange {
    background-color: rgb(255, 213, 80);
  }

  .num {
    display: inline-block;
    background: url('../assets/imgs/examTitle.png') no-repeat 100% 100%;
    background-size: contain;
    height: 37px;
    width: 37px;
    line-height: 30px;
    color: #fff;
    font-size: 20px;
    text-align: center;
    margin-right: 15px;
  }
</style>
