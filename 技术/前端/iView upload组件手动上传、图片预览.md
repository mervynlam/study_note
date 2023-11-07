# iView upload组件手动上传、图片预览

## 手动上传

[iview文档](https://www.iviewui.com/components/upload#SDSC)中并没有实际的上传操作

![image-20211008172847903](https://mervyn-markdown-images.oss-cn-beijing.aliyuncs.com/202311071627744.png)

**实现**

```vue
<Upload
        ref="upload"
        :format="['jpg','jpeg','png']"
        :before-upload="handleUpload"
        type="drag"
        :show-upload-list="false"
        :action="actionUrl"
        :data="uploadData">
    <Button icon="ios-cloud-upload-outline">选择文件</Button>
```

```javascript
handleUpload(file) {
    this.file = file
    return false
    //通过return false阻止上传流程
},
upload() {
    //附带参数
    this.uploadData.id=1
    this.uploadData.type=3
    //通过refs上传文件
    this.$refs.upload.post(this.file)
}
```

![image-20211008173306727](https://mervyn-markdown-images.oss-cn-beijing.aliyuncs.com/202311071627909.png)

## 手动上传图片展示

```vue
<FormItem label="图片" prop="image">
    <div>
        <img v-if="uploadFiles.image" :src="uploadFiles.image">
    </div>
    <Upload
            ref="upload"
            :format="['jpg','jpeg','png']"
            :before-upload="handleUpload"
            type="drag"
            :show-upload-list="false"
            :action="actionUrl"
            :data="uploadData">
        <Button icon="ios-cloud-upload-outline">选择文件</Button>
    </Upload>
</FormItem>
```

```javascript
handleUpload(file) {
    this.file = file

    const _this = this
    const reader = new FileReader()
    reader.readAsDataURL(file);
    reader.onload = function (e) {
        // 文件base64
        const fileBase64 = this.result
        _this.uploadFiles.image = fileBase64
    }
    //阻止上传流程
    return false
},
```

**效果**

![image-20211008174010641](https://mervyn-markdown-images.oss-cn-beijing.aliyuncs.com/202311071627712.png)

![20211008174106](https://mervyn-markdown-images.oss-cn-beijing.aliyuncs.com/202311071627826.jpg)

## 参考资料

[iview upload爬坑 之手动上传以及动态修改附带参数 附后台接受测试代码](https://blog.csdn.net/w779050550/article/details/105279790)

