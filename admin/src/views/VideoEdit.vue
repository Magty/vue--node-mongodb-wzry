<template>
  <div class="about">
     <!-- 导航区 -->
    <Breadcrumb :breadcrumbItem="breadcrumbItem"></Breadcrumb>
    <el-card>
    <el-form label-width="80px" @submit.native.prevent="save">
      <el-form-item label="标题">
        <el-input v-model="model.title" clearable></el-input>
      </el-form-item>
        <el-form-item label="封面图">
            <el-upload class="avatar-uploader" 
            :action="upLoadUrl" 
            :headers="getAuthHeaders()" 
            :show-file-list="false"
              :on-success="res => model.image = res.url" 
              :before-upload="beforeAvatarUpload">
              <img v-if="model.image" :src="model.image" class="avatar">

              <i v-else class="el-icon-plus avatar-uploader-icon"></i>
            </el-upload>
          </el-form-item>
             <el-form-item label="所属分类">
            <el-select v-model="model.categories" multiple>
              <el-option v-for="item of categories" :key="item._id" :label="item.name" :value="item._id"></el-option>
            </el-select>
          </el-form-item>
      <el-form-item label="视频">
      <el-upload
        class=""
        :action="upLoadUrl" 
        :headers="getAuthHeaders()" 
        :show-file-list="false"
        :on-success="res => model.video = res.url" 
        drag
        multiple>
        <video v-if="model.video" :src="model.video" width="355" controls></video>
  <i class="el-icon-upload"></i>
  <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
</el-upload>
      </el-form-item>
      <el-form-item label="投稿">
        <el-input v-model="model.submit" clearable></el-input>
      </el-form-item>
      <el-form-item label="粉丝">
        <el-input v-model="model.fan" clearable></el-input>
      </el-form-item>
      <el-form-item label="观看次数">
        <el-input v-model="model.see" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" style="margin-left:5rem;" native-type="submit">保存</el-button>
      </el-form-item>
    </el-form>
    </el-card>
  </div>
</template>
<script>
import Breadcrumb from '../components/Breadcrumb'
  export default {
    props: {
      id: {}
    },
     components: {
      Breadcrumb
    },
    data() {
      return {
        model: {
          title:'',
          image:'',
          video:'',
          submit:'',
          fan:'',
          see:'',
        },
        categories: [],
        breadcrumbItem: ['运营管理', '视频管理', `${this.id ? '编辑视频':'新建视频'}`],
      };
    },
    methods: {
        afterUpload(res) {
        this.model.image = res.url
      },
      beforeAvatarUpload(file) {
        const isJPG = file.type === 'image/jpeg';
        const isLt2M = file.size / 1024 / 1024 < 2;
        if (!isJPG) {
          this.$message.error('上传头像图片只能是 JPG 格式!');
        }
        if (!isLt2M) {
          this.$message.error('上传头像图片大小不能超过 2MB!');
        }
        return isJPG && isLt2M;
      },
      // 编辑/保存数据
      async save() {
        let res
        if (this.id) {
          res = await this.$http.put(`rest/videos/${this.id}`, this.model);
          this.model = res.data;
        } else {
          res = await this.$http.post('rest/videos', this.model)
          this.model = res.data;
        }
        this.$router.push('/videos/list')
        if (this.id) {
          this.$message({
            type: 'success',
            message: '修改成功'
          })
        } else {
          this.$message({
            type: 'success',
            message: '保存成功'
          })
        }
      },
      // 向后台请求需要编辑的数据
      async fetch() {
        const res = await this.$http.get(`rest/videos/${this.id}`);
        this.model = res.data;
      },
       // 获取视频分类列表（后期做优化）
      async getCategories() {
        const res = await this.$http.get(`rest/categories`);
        this.categories = res.data;
      },
        async handleImageAdded(file, Editor, cursorLocation, resetUploader) {
        const formData = new FormData();
        formData.append("file", file);
        const res = await this.$http.post("upload", formData);
        Editor.insertEmbed(cursorLocation, "image", res.data.url);
        resetUploader();
      }
    },
    created() {
      this.id && this.fetch(); //   
      this.getCategories()     
    }
  };
</script>
<style>
/* // 富文本编辑器里面的播放器样式 */
  .video-container-style iframe{
    width: 100%;
    height: auto;
  }
</style>