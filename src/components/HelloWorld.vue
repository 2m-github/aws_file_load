<template>
  <v-container>
    <h1>파일 리스트</h1>
    <ul>
      <li v-for="(item,index) in imgFileList" :key="index">
        {{index + 1}}. {{item.Key}} {{item.LastModified}}
        <v-btn @click="deleteImgFile(item.Key)">Delete</v-btn>
      </li>
    </ul>
    <h1>파일 업로드</h1>
    <div><img :src="imgFileURL" alt=""></div>
    <input id="file-selector" ref="imgFile" type="file" @change="selectFile()" />
    <v-btn color="primary" @click="uploadFile">upload</v-btn>
  </v-container>
</template>
<script>
import AWS from 'aws-sdk'
export default {
  data(){
    return {
      imgFileList:[],
      imgFile: null,
      imgFileURL: null,
      albumBucketName: "photo-2m",
      bucketRegion: "ap-northeast-2",
      IdentityPoolId: "ap-northeast-2:2c6a4897-21c1-4594-be2b-77874900c577"
    }
  },
  created(){
    this.listAlbums();
  },
  methods: {
    selectFile(){
      this.imgFile = this.$refs.imgFile.files[0]
      let reader = new FileReader();
      reader.readAsDataURL(this.imgFile)
      reader.onload = ()=>{
        this.imgFileURL = reader.result
      }
      console.log('-------------',window.URL.createObjectURL(this.imgFile))

      
      
    },
    // upload
    uploadFile(){
      AWS.config.update({
        region: this.bucketRegion,
        credentials: new AWS.CognitoIdentityCredentials({
          IdentityPoolId: this.IdentityPoolId
        })
      });

      var s3 = new AWS.S3({
        apiVersion: "2006-03-01",
        params: { Bucket: this.albumBucketName }
      });

      // Use S3 ManagedUpload class as it supports multipart uploads
      let photoKey = this.imgFile.name
      var upload = new AWS.S3.ManagedUpload({
        params: {
          Bucket: this.albumBucketName,
          Key: photoKey,
          Body: this.imgFile
        }
      });

      var promise = upload.promise();

      promise.then(
        data => {
          console.log("Successfully uploaded photo.");
          this.listAlbums()

        },
        err => {
          return alert("There was an error uploading your photo: ", err.message);
        }
      );

    },
    //list
    listAlbums(){
      AWS.config.update({
        region: this.bucketRegion,
        credentials: new AWS.CognitoIdentityCredentials({
          IdentityPoolId: this.IdentityPoolId
        })
      });

      var s3 = new AWS.S3({
        apiVersion: "2006-03-01",
        params: { Bucket: this.albumBucketName }
      });

      s3.listObjects({ Delimiter: "/" }, (err, data) => {
        if (err) {
          return alert("There was an error listing your albums: " + err.message);
        } else {
          data.Contents.map(e => {
            let date = new Date(e.LastModified);
            let year = date.getFullYear();
            let month = date.getMonth() + 1;
            month = month >= 10 ? month : '0' + month
            let day = date.getDate();
            day = day >= 10 ? day : '0' + day;
            let h = date.getHours();
            h = h >= 10 ? h : '0' + h;
            let m = date.getMinutes();
            m = m >= 10 ? m : '0' + m;
            let s = date.getSeconds();
            s = s >=10 ? s : '0' + s;
            const fullTime = year + "-" + month + "-" + day + " " + h + ":" + m + ":" + s;
            e.LastModified = fullTime
            console.log('t==',e.LastModified)
            this.imgFileList.push(e)
          })
          console.log('data',this.imgFileList)
        }
      });
    },

    //Delete
    deleteImgFile(key){
      AWS.config.update({
        region: this.bucketRegion,
        credentials: new AWS.CognitoIdentityCredentials({
          IdentityPoolId: this.IdentityPoolId
        })
      });

      var s3 = new AWS.S3({
        apiVersion: "2006-03-01",
        params: { Bucket: this.albumBucketName }
      });

      s3.deleteObject(
        {
          //Delete: { Objects: key, Quiet: true }
          Key: key
        },
        (err, data) => {
          if (err) {
            return alert("There was an error deleting your album: ", err.message);
          }
          console.log("Successfully deleted album.");
          this.listAlbums();
        }
      );
    }


  }
}
</script>