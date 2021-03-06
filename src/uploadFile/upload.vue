<style lang="scss">
    /*前缀*/
    $prefixCls: 'upload-file-plugin';

    .#{$prefixCls}{

        background-color: #ffffff;

        .add-file{

            .size-tip{
                font-size: 12px;
                color: #90A2B9;
                line-height: 12px;
            }
        }

        .file-list{
            margin-top: 8px;

            .file-item{
                display: flex;
                position: relative;
                max-width: 750px;
                font-size: 14px;
                height: 30px;
                line-height: 30px;
                overflow: hidden;
                cursor: pointer;

                &:hover{
                    background: #F5F7F9;
                }

                .name{
                    flex: 1;
                    overflow: hidden;
                    white-space: nowrap;
                    text-overflow: ellipsis;
                    padding-left: 9px;

                    .text{
                        padding-left: 5px;
                        color: #657180;
                    }
                }

                .size{

                }

                .status{
                    width: 100px;
                    text-align: center;

                    .success{
                        display: none;
                        color: #1BBE6B;
                        i{font-size: 20px;}
                    }

                    .uploading{
                        color: #657180;
                    }

                    .error{
                        color: #FF5557;
                        display: none;
                    }
                }

                .handler{
                    width: 100px;
                    padding-right: 9px;

                    a{
                        color: #848F9D;

                        &:hover{
                            color: #657180;
                        }
                    }

                    .restart{
                        display: none;
                    }
                }

                .size, .handler{
                    text-align: right;
                }

                .progress{
                    position: absolute;
                    height: 30px;
                    opacity: 0.3;
                    background: #77AEFF;
                }

            }

            .success{

                .name{
                    .text{color: #2D8CF0;}
                }

                .status{
                    .success{
                        display: block;
                    }

                    .uploading{
                        display: none;
                    }

                    .error{
                        display: none;
                    }
                }
            }

            .uploading{
                .status{
                    .success{
                        display: none;
                    }

                    .uploading{
                        display: none;
                    }

                    .error{
                        display: none;
                    }
                }
            }

            .waiting{
                .status{
                    .success{
                        display: none;
                    }

                    .uploading{
                        display: block;
                    }

                    .error{
                        display: none;
                    }
                }
            }

            .error{

                .status{

                    .success{
                        display: none;
                    }

                    .uploading{
                        display: none;
                    }

                    .error{
                        display: block;
                    }
                }

                .handler{
                    .restart {
                        display: inline-block;
                    }
                }
            }
        }

        /* margin-left: 20px */
        .m-l-20{
            margin-left: 20px;
        }

    }

    /* 选择文件 */
    .select-file-body{
        margin: -16px;
        cursor: default;

        .select-file-type{
            position: relative;
            height: 60px;
            line-height: 60px;
            text-align: center;
            border-bottom: 1px solid #E9EAEC;

            &:hover{
                background: #F5F7F9;
            }

            .file{
                display: block;
                position: absolute;
                width: 100%;
                height: 100%;
                opacity: 0;
                top: 0;
                left: 0;
                bottom: 0;
                right: 0;
                z-index: 1;
            }
        }
    }

    .select-file-footer{
        margin-top: 20px;
        margin-bottom: 20px;

        button{
            width: 260px;
        }
    }
</style>

<template>
    <div :class="[prefixCls]">
        <div class="add-file" v-if="showAddBtn">
            <Button type="ghost" @click="showFileSelect = true">添加附件</Button>
            <span class="size-tip">附件不超过{{maxSizeText}}</span>
        </div>
        <div class="file-list">
            <template v-for="(item, index) in fileList">
                <file-item
                    :data="item"
                    :index="index"
                    :readonly="readonly"

                    @item-click="itemClick"
                    @restart="restart"
                    @del="del"
                    @save-cloud="saveCloud"
                    @download="download"
                ></file-item>
            </template>
        </div>
        <Modal
            v-if="!readonly"
            ref="fileSelectType"
            v-model="showFileSelect"
            :width="400"
            title="选择上传附件方式">
            <div class="select-file-body">
                <div class="select-file-type">
                    从本地选择
                    <input type="file" title=" " class="file" multiple @change="localUpload">
                </div>
                <div class="select-file-type" @click="cloudUpload">
                    从网盘选择
                </div>
            </div>
            <div slot="footer" class="select-file-footer">
                <Button type="ghost" @click="showFileSelect = false">取消</Button>
            </div>
        </Modal>
        <!-- 图片预览 -->
        <image-preview
            :src="previewImgSrc"
            v-model="showPreviewImg"
        ></image-preview>
        <!-- 附件预览 -->
        <file-preview
            :uuid="previewFileUuid"
            :fileName="previewFileName"
            v-model="showPreviewFile"
        ></file-preview>
        <!-- 选择网盘 -->
        <cloud
            v-if="!readonly"
            v-model="openCloud"
            @selected="selectedCloudFile"
        ></cloud>
        <!-- 保存至网盘 -->
        <save-cloud
            v-if="readonly"
            :selected-file-id="selectedFileId"
            v-model="openSaveToCloud"
        ></save-cloud>
    </div>
</template>

<script>
    import lib from './lib/index'
    import ImagePreview from '../imagePreview/index'
    import FilePreview from '../filePreview/index'
    import FileItem from './item.vue'
    import Ajax from './Ajax'

    import Cloud from './cloud/cloud.vue'
    import SaveCloud from './cloud/saveCloud.vue'
    import Utils from './Utils'

    let {BrowserMd5File, CosCloud} = lib;

    export default {
        components: {FileItem, ImagePreview, FilePreview, Cloud, SaveCloud},
        props: {
            value: {
                type: Object,
                default(){
                    return {};
                }
            }
            /** 文件uuid */
            , uuids: {
                type: Array,
                default(){
                    return [];
                }
            }
            /** 单文件大小默认10M */
            , maxsize: {
                type: Number,
                default: 10
            }
            /** 上传文件个数默认不限制 */
            , maxlength: {
                type: Number,
                default: -1
            }
            /** 是否只读状态 */
            , readonly: {
                type: Boolean,
                default: false
            }
        },
        data(){
            return {
                prefixCls: 'upload-file-plugin'
                // 选择文件
                , showFileSelect: false
                // 打开网盘文件
                , openCloud: false
                // 保存至网盘
                , openSaveToCloud: false
                // 文件列表
                , fileList: []
                // 状态
                , status: {
                    success: 'success'
                    , uploading: 'uploading'
                    , error: 'error'
                    , waiting: 'waiting'
                }
                // 单个文件大小
                , oneFileMaxSize: 0
                // 同时上传个数
                , maxUploading: 5
                // 当前上传数
                , uploading: 0
                // 是否触发上传成功事件
                , isEmitUploadSuccess: false

                // 图片预览url
                , previewImgSrc: ''
                , showPreviewImg: false
                // 附件预览
                , previewFileUuid: ''
                , previewFileName: ''
                , showPreviewFile: false
                // 当前选择文件id
                , selectedFileId: ''
            };
        },
        computed: {
            /** 等待上传文件 */
            waitingFiles(){
                let t = this;
                let status = t.status;
                return t.fileList.filter((item)=>{
                    return item.status === status.waiting;
                });
            },
            /** 最大大小 */
            maxSizeText(){
                return Utils.formatFileSize(this.oneFileMaxSize);
            },
            showAddBtn(){
                let t = this;
                return !t.readonly && (t.maxlength === -1 || t.fileList.length < t.maxlength);
            }
        },
        created(){
            let t = this;

            t.oneFileMaxSize = t.maxsize * 1048576;

            if(t.uuids && t.uuids.length > 0){
                t.loadFile();
            }
        },
        methods: {
            /**
             *  本地上传
             * @param e
             */
            localUpload(e){
                let t = this;
                t.showFileSelect = false;
                let target = e.target;
                let files = target.files;
                t.addFile(files);
                target.value = '';
            }
            /**
             * 网盘上传
             */
            , cloudUpload(){
                let t = this;
                t.showFileSelect = false;
                t.openCloud = true;
            }
            /** 选中网盘文件 */
            , selectedCloudFile(fileList){
                let t = this;
                let status = t.status;
                fileList.forEach(item=>{
                    let size = item.fileSize;
                    let name = item.fileName;
                    let icon = Utils.getIcon(name);
                    t.fileList.push({
                        name: name,
                        size: size,
                        sizeText: Utils.formatFileSize(size),
                        type: icon.type,
                        icon: icon.icon,
                        md5: item.fileMd5, // 网盘返回文件md5
                        uuid: '',
                        src: '',
                        status: status.success,
                        process: 0,
                        loading: false,
                        isNew: true
                    });
                });
            }
            /**
             * 行点击
             * */
            , itemClick(index, data){
                let t = this;
                let item = t.fileList[index];
                let name = item.name;
                // 只有上传成功文件才能预览
                if(item.status !== t.status.success){
                    return ;
                }
                let ext = name.substring(name.lastIndexOf('.') + 1);
                if(item.type === 2){
                    // 图片预览
                    if(ext !== 'psd'){
                        t.showPreviewImg = true;
                        if(!item.src){
                            Ajax.getImgUrl(item.uuid).then(function(data){
                                item.src = data;
                                t.previewImgSrc = item.src;
                            });
                        }else{
                            t.previewImgSrc = item.src;
                        }
                    }else{
                        this.$Message.warning('该文件类型不支持预览！');
                    }
                }else{
                    // 文件预览
                    if(t.oneOf(ext, ['doc','docx','rtf','xls','xlsx','et','pptx','ppt','pdf','wps','txt', 'rar', 'zip', '7z'])){
                        t.previewFileUuid = item.uuid;
                        t.previewFileName = item.name;
                        t.showPreviewFile = true;
                    }else{
                        this.$Message.warning('该文件类型不支持预览！');
                    }
                }

            }
            /**
             * 重试
             **/
            , restart(index){
                let t = this;
                let item = t.fileList[index];
                if(item){
                    item.status = t.status.waiting;
                }
            }
            /**
             * 删除
             * */
            , del(index){
                let t = this;
                let item = t.fileList[index];
                if(item && item.status === t.status.success){
                    this.$Modal.confirm({
                        title: '提示',
                        content: '<p style="text-align: center">您确定要删除附件吗？</p>',
                        onOk: () => {
                            t.isEmitUploadSuccess = false;
                            let removeFile = t.fileList.splice(index, 1);
                            Ajax.delFile([removeFile[0].uuid]).then(()=>{
                                t.$emit('remove-file', removeFile);
                            });
                        }
                    });
                }else{
                    t.fileList.splice(index, 1);
                }
            }
            /**
             * 保存到网盘
             * */
            , saveCloud(index){
                let t = this;
                let item = t.fileList[index];
                if(item){
                    t.selectedFileId = item.uuid;
                    t.openSaveToCloud = true;
                }
            }
            /**
             * 下载
             * */
            , download(index){
                let t = this;
                let item = t.fileList[index];
                if(item){
                    Ajax.getImgUrl(item.uuid).then(function(data){
                        window.location.href = data;
                    });
                }
            }
            /**
             * 添加文件
             * */
            , addFile(fileList){
                let t = this;
                let status = t.status;
                let readyLen = t.fileList.length;
                let len = fileList.length;
                // 上传文件数限制
                if(t.maxlength !== -1 && len > (t.maxlength - readyLen)){
                    this.$Modal.warning({
                        title: '提示',
                        content: '<p style="text-align: center;">最多只能上传' + t.maxlength + '个文件</p>'
                    });
                    return ;
                }
                let hasExceedSize = false;
                for(let i = 0; i<len; i++){
                    let file = fileList[i];
                    let size = file.size;
                    let name = file.name;
                    if(t.oneImgMaxSize < size){
                        hasExceedSize = true;
                        continue;
                    }
                    let icon = Utils.getIcon(name);
                    t.fileList.push({
                        file: file,
                        name: name,
                        size: size,
                        sizeText: Utils.formatFileSize(size),
                        type: icon.type,
                        icon: icon.icon,
                        uuid: '',
                        src: '',
                        status: status.waiting,
                        process: 0,
                        loading: false,
                        isNew: true
                    });
                }

                if(hasExceedSize){
                    this.$Modal.warning({
                        title: '提示',
                        content: '<p style="text-align: center;">单个文件最大不超过' + t.maxSizeText + '</p>'
                    });
                }
            }
            /**
             * 添加队列
             *
             * @param item
             */
            , addQueue(item){
                let t = this;
                if(!item){return ;}
                if(t.uploading < t.maxUploading) {

                    t.uploading++;
                    item.status = t.status.uploading;

                    t.md5(item);
                    t.addQueue(t.waitingFiles[0]);
                }
            }
            /**
             * 文件计算md5值
             */
            , md5(item){
                let t = this;
                if(item.md5){
                    // 选网盘文件
                    t.preupLoadFile(item);
                }else {
                    // 选本地文件
                    item.sparkMd5 = BrowserMd5File(item.file, function (err, md5) {
                        if (err) {
                            t.completeUpload(false, item);
                            return;
                        }

                        item.md5 = md5;

                        t.preupLoadFile(item);
                    });
                }
            }
            /**
             *  预上传
             *
             *  @param file
             * */
            , preupLoadFile(file){
                let t = this;
                // 1、验证文件在服务器是否存在
                Ajax.preupLoadFile(file.name, file.size, file.md5).then( data =>{
                    if(!data){
                        t.completeUpload(false, file);
                        return ;
                    }
                    if (data && data.uploadFlg === 'true') {
                        file.uuid = data.data.uuid;
                        // 2、秒存
                        t.completeUpload(true, file);
                    } else {
                        // 3、文件在服务器不存在，则需要上传云
                        t.getSign(file, data);
                    }
                });
            }
            /**
             * 获取签名
             *
             * @param file
             * @param upFileName
             */
            , getSign(file, upFileName){
                let t = this;
                Ajax.getSign(upFileName, file.name).then((data) => {
                    t.uploadCossh(data, file);
                });
            }
            /**
             * 上传腾讯云
             *
             * @param signInfo
             * @param file
             */
            , uploadCossh(signInfo, file){
                let t = this;
                let cloudSign = Ajax.cloudSign;
                // 创建上传云服务器对象
                file.cos = new CosCloud({
                    appid: cloudSign.appid,// APPID 必填参数
                    bucket: cloudSign.bucketName,//bucketName 必填参数
                    region: cloudSign.region,//地域信息 必填参数 华南地区填gz 华东填sh 华北填tj
                    getAppSign: function (callback) {
                        callback(signInfo.sign);
                    },
                    getAppSignOnce: function (callback) {
                        callback(signInfo.sign);
                    }
                });

                file.cos.uploadFile(function(data){
                    // 成功回调
                    t.sendCosshResult(data, file);
                }, function(data){
                    // 失败回调
                    if(data.statusText !== 'abort'){
                        t.completeUpload(false, file);
                    }
                }, function(percent, sha1Check){
                    // 上传进度
                    t.fileUploadProgress(percent, sha1Check, file);
                }, cloudSign.bucketName, signInfo.cosPath, file.file, 1, function(taskId){
                    file.taskId = taskId;
                });

            }
            /**
             * 发送上传云服务成功反馈信息到后台
             *
             * @param data
             * @param file
             */
            , sendCosshResult(data, file){
                let t = this;
                let responseJson = typeof data === "string" ? JSON.parse(data): data;
                Ajax.upLoadFile(file.name, file.size, file.md5, responseJson).then( data => {
                    file.uuid = data.uuid;
                    // 文件上传完成
                    t.completeUpload(true, file);
                });
            }
            /**
             * 文件上传进度
             * */
            , fileUploadProgress(percent, sha1Check, file){
                let t = this;
//                let currTime = new Date().getTime();
//                let currLoaded = percent * file.size;

//                let dT = currTime - file.lastTime;
//                let dL = currLoaded - file.lastLoaded;

//                let speed = Math.abs(parseInt(dL / dT));

//                file.lastLoaded = currLoaded;
//                file.lastTime = currTime;

                let progress = parseInt(percent * 100, 10) + '%';
                file.process = progress;
//                t.file.statusDesc = progress + '('+ (speed > 1000 ? (speed / 1024).toFixed(2) + 'M': speed + 'KB') +'/s)';
            }
            /**
             * 上传完成: true完成；false失败
             *
             * @param type
             * @param file
             */
            , completeUpload(type, file){
                let t = this;
                if(type){
                    file.status = t.status.success;
                }else{
                    file.status = t.status.error;
                }
                // 不管成功失败，都算成功
                t.uploading--;
                file.process = 0;

                t.getValue();
            }
            /**
             * 数组包含某个值
             * */
            , oneOf (value, validList) {
                for (let i = 0; i < validList.length; i++) {
                    if (value === validList[i]) {
                        return true;
                    }
                }
                return false;
            }
            /**
             * 加载文件
             * */
            , loadFile(){
                let t = this;
                let status = t.status;

                Ajax.getFileList(t.uuids).then((data)=>{
                    let list = [];
                    data.forEach(item=>{
                        let name = item.fileName;
                        let size = item.fileSize;
                        let icon = Utils.getIcon(name);
                        list.push({
                            name: name,
                            size: size,
                            sizeText: Utils.formatFileSize(size),
                            type: icon.type,
                            icon: icon.icon,
                            uuid: item.uuid,
                            src: '',
                            status: status.success,
                            process: 0,
                            loading: false,
                            isNew: false
                        });
                    });
                    t.fileList = list;
                });
            }
            /**
             * 获取最终结果
             */
            , getValue(){
                let t = this;
                let status = t.status;
                let files = [], uuids = [], isSuccess = true;
                t.fileList.forEach(item=>{
                    files.push({
                        name: item.name,
                        size: item.size,
                        status: item.status,
                        uuid: item.uuid,
                        isNew: item.isNew
                    });
                    if(item.status === status.success){
                        uuids.push(item.uuid);
                    }else{
                        isSuccess = false;
                    }
                });

                t.$emit('input', {
                    uuids: uuids,         // 所有文件uuid
                    files: files,         // 文件列表信息
                    upLoading: !isSuccess // 是否上传中
                });

                // 全部上传完成
                if(t.isEmitUploadSuccess && isSuccess){
                    t.$emit('upload-success', uuids, files);
                }

                t.isEmitUploadSuccess = true;
            }
        },
        watch: {
            waitingFiles(val){
                this.addQueue(val[0]);
            },
            uuids(val){
                let t = this;
                if(val && val.length > 0){
                    t.isEmitUploadSuccess = false;
                    t.loadFile();
                }else{
                    t.fileList = [];
                }
            },
            fileList(val){
                this.getValue();
            },
            showPreviewFile(val){
                if(!val){
                    this.previewFileUuid = '';
                    this.previewFileName = '';
                }
            }
        },
        mounted(){
            let t = this;
            if(t.$refs.fileSelectType) {
                let $el = t.$refs.fileSelectType.$el;
                $el.querySelector('.ivu-modal-wrap').style.zIndex = 999999;
                $el.querySelector('.ivu-modal-mask').style.zIndex = 999998;
            }
        },
        destroyed(){

        }
    }
</script>
