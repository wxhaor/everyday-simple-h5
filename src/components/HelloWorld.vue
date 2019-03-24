<template xmlns:v-clipboard="http://www.w3.org/1999/xhtml">
  <div class="hello">
    <el-form :model="form" inline label-width="100px" class="demo-form-inline">
      <el-form-item label="文章名称">
        <el-autocomplete
          class="inline-input"
          v-model="form.articleName"
          value-key="articleName"
          :fetch-suggestions="querySearch"
          placeholder="模糊匹配文章名称"
          :trigger-on-focus="false"
          @select="handleSelect"
        ></el-autocomplete>
      </el-form-item>
      <el-form-item label="文章id">
        <el-input v-model="form.id" placeholder="文章id" disabled></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="onSubmit">添加文章</el-button>
      </el-form-item>
    </el-form>
    <el-upload
      class="upload-demo"
      drag
      :data="form"
      :action="uploadServer"
      :on-success="uploadOnSuccess"
      :on-change="uploadOnChange"
      :show-file-list="false"
      multiple>
      <i class="el-icon-upload"></i>
      <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
      <!--<div class="el-upload__tip" slot="tip">只能上传jpg/png文件，且不超过500kb</div>-->
    </el-upload>

    <div class="img-list">
      <div class="img-content" v-for="(item,key) in fileList" :key="key">
        <img :src="item.imageUrl">
        <div class="name">
          <div>{{ item.imageName }}</div>
          <el-button type="text" v-clipboard:copy="item.imageUrl" v-clipboard:success="successMessage"
                     v-clipboard:error="errorMessage">复制链接
          </el-button>
        </div>
        <!-- 删除icon -->
        <div class="del">
          <i @click="handleFileRemove(item,key)" v-clipboard:copy="'[]('+item.imageUrl+')'"
             v-clipboard:success="copySuccessMessage" class="el-icon-document"></i>
        </div>
        <!-- 放大icon -->
        <div class="layer" @click="handleFileEnlarge(item.imageUrl)">
          <i class="el-icon-view"></i>
        </div>
      </div>
      <div v-if="!pass && progress !== 0" class="img-content img-progress">
        <el-progress type="circle" :percentage="progress" :status="proStatus"></el-progress>
      </div>
    </div>
    <el-dialog title="" :visible.sync="isEnlargeImage" size="large" :modal-append-to-body="false" top="8%" width="60%">
      <img @click="isEnlargeImage = false" style="width:100%;" :src="enlargeImage">
    </el-dialog>
  </div>

</template>

<script>
  export default {
    data() {
      return {
        form: {
          articleName: "",
          id: ""
        },
        restaurants: [],//文章列表建议
        uploadServer: "http://localhost:8082/blog/upload",
        isEnlargeImage: false,//放大图片
        enlargeImage: '',//放大图片地址
        progress: 0,//上传进度
        pass: null,//是否上传成功
        fileList: []//文件列表
      }
    },
    methods: {
      querySearch(queryString, cb) {
        const h = this.$createElement;
        let self = this;
        self.form.id = ""
        self.fileList=[]
        self.url = 'http://localhost:8082/blog/findArticle';
        self.$axios.post(self.url, self.form).then((res) => {
          self.restaurants = res.data;
        }).then((res) => {
          // 调用 callback 返回建议列表的数据
          cb(self.restaurants);
          if(self.restaurants.length === 0){
            this.$notify({
              title: '搜索结果',
              message: '没有搜索到相关文章图片,可以选择添加该文章',
              position: 'bottom-right',
              duration: 3000
            });
          }else{
            this.$notify({
              title: '搜索结果',
              message: '搜索到相关结果为'+ self.restaurants.length + '条',
              position: 'bottom-right',
              duration: 3000
            });
          }
        })
      },
      handleSelect(item) {
        let self = this;
        self.form = item;

        self.url = 'http://localhost:8082/blog/findImageByArticleId';
        self.$axios.post(self.url, self.form).then((res) => {
          self.fileList = res.data;
        });
      },
      onSubmit() {
        let self = this;
        if (self.form.id != '') {
          this.$message.error("请勿重复新增")
          return
        }

        self.url = 'http://localhost:8082/blog/addArticle';
        self.$axios.post(self.url, self.form).then((res) => {
          self.form = res.data;
        });

        console.log('submit!');
      },
      uploadOnSuccess(response, file, fileList) {
        this.fileList.push({imageName: file.name, imageUrl: response});

      },
      uploadOnChange(file) {
        // console.log(file)
        if (file.status == 'ready') {
          console.log("ready")
          //重置progress组件
          this.pass = null;
          this.progress = 0;
        } else if (file.status == 'fail') {
          this.$message.error("图片上传出错，请刷新重试！")
        }
      },
      successMessage(e) {
        this.$message('复制成功!');
      },
      errorMessage(e) {
        this.$message({
          message: '复制失败!',
          type: 'warning'
        });
      },
      handleFileEnlarge(_url) {//放大图片
        console.log(_url)
        if (_url) {
          this.enlargeImage = _url;
          this.isEnlargeImage = !this.isEnlargeImage;
        }
      },
      copySuccessMessage(e) {
        this.$message('md格式url复制成功!');
      },
      handleFileRemove(file, i) {//删除图片
        console.log(file, i)
      }

    }
  }
</script>

<style scoped>
  * {
    box-sizing: border-box;
  }

  .img-list {
    overflow: hidden;
    width: 100%;
  }

  .img-list .img-content {
    float: left;
    position: relative;
    display: inline-block;
    width: 200px;
    height: 270px;
    padding: 5px;
    margin: 5px 20px 20px 0;
    border: 1px solid #d1dbe5;
    border-radius: 4px;
    transition: all .3s;
    box-shadow: 0 2px 4px 0 rgba(0, 0, 0, .12), 0 0 6px 0 rgba(0, 0, 0, .04);
  }

  .img-list .img-content img {
    display: block;
    width: 100%;
    height: 190px;
    margin: 0 auto;
    border-radius: 4px;
  }

  .img-list .img-content .name {
    margin-top: 10px;
  }

  .img-list .img-content .name > div {
    width: 90%;
    text-overflow: ellipsis;
    overflow: hidden;
    height: 25px;
    line-height: 25px;
  }

  .img-list .img-content:hover .del,
  .img-list .img-content:hover .layer {
    opacity: 1;
  }

  .img-list .img-content .del,
  .img-list .img-content .layer {
    opacity: 0;
    transition: all .3s;
  }

  .img-list .img-content .del {
    position: absolute;
    bottom: 10px;
    right: 10px;
    color: #8492a6;
    cursor: pointer;
    font-size: 1.1em;
  }

  .img-list .img-content .layer {
    position: absolute;
    left: 0;
    right: 0;
    top: 0;
    height: 200px;
    color: #fff;
    text-align: center;
    z-index: 5;
    background-color: rgba(0, 0, 0, .4);
  }

  .img-list .img-content .layer i {
    font-size: 1.6em;
    margin-top: 80px;
  }

  .img-list .img-progress {
    text-align: center;
    padding-top: 50px;
  }
</style>
