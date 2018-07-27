<template>
    <div>
        <div class="container">
            <el-form ref="form" :model="form" label-width="80px">
                <el-form-item label="时间">
                    <el-date-picker
                        v-model="form.date"
                        type="datetime"
                        value-format="yyyy-MM-dd HH:mm:ss"
                        placeholder="选择日期时间">
                    </el-date-picker>
                </el-form-item>
                <el-form-item label="标题">
                    <el-input v-model="form.title"></el-input>
                </el-form-item>
                <el-form-item label="类型">
                    <el-select v-model="form.type" placeholder="请选择文章类型">
                        <el-option
                            v-for="item in options"
                            :key="item.label"
                            :label="item.label"
                            :value="item.label">
                        </el-option>
                    </el-select>
                </el-form-item>
            </el-form>
            <!--{{form.content}}-->
            <quill-editor class="ui-editor" ref="myTextEditor" v-model="form.content" :options="editorOption"></quill-editor>
            <el-button class="editor-btn" type="primary" @click="submit">提交</el-button>
            <el-upload
                class="upload-demo"
                :action="url"
                :on-remove="handleRemove"
                :on-success="upSuccess"
                :file-list="fileList"
                :on-error="upError"
                list-type="picture">
                <el-button size="small" type="primary">上传图片</el-button>
                <!--<div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>-->
            </el-upload>
        </div>
    </div>
</template>

<script>
    import 'quill/dist/quill.core.css';
    import 'quill/dist/quill.snow.css';
    import 'quill/dist/quill.bubble.css';
    import { quillEditor } from 'vue-quill-editor';
    export default {
        data: function(){
            return {
                //图片列表
                fileList: [],
                editorOption: {
                    placeholder: 'Hello World'
                },
                form: {
                    date:"", //时间
                    title: "", //标题
                    type:"", //类型
                    content: '', //内容
                    id:this.$route.query.id //编辑文章的时候用
                },
                //文章类型
                options:[
                    {
                        label:"js"
                    },
                    {
                        label:"html"
                    },
                    {
                        label:"css"
                    }
                ],
                url: sessionStorage.getItem("url") + "/upload" //上传地址
            }
        },
        components: {
            quillEditor
        },
        methods: {
            //文章提交
            submit () {
                //编辑过来的
                if(this.$route.query.id){
                    this.$http.post("/updateArticle",{
                        "form": this.form
                    },(data)=>{
                        this.$message({
                            message: "修改成功！",
                            type: 'success',
                            duration: '1500',
                            center: true
                        });
                    })
                }else{
                    //新文章提交
                    this.$http.post("/article",{
                        "form": this.form
                    },(data)=>{
                        this.$message({
                            message: "发布成功！",
                            type: 'success',
                            duration: '1500',
                            center: true
                        });
                    });
                }
            },
            //去重函数
            removeByValue(arr, val) {
                for(var i=0; i<arr.length; i++) {
                    if(arr[i].url == val) {
                        arr.splice(i, 1);
                        return arr;
                    }
                }
            },
            //图片删除
            handleRemove(file, fileList) {
                this.$http.post("/remove",{
                    url: file.url
                }, req => {
                    this.$message({
                        message: '删除成功！',
                        type: 'success'
                    });
                    this.fileList = this.removeByValue(this.fileList,file.url);
                });
            },

            //上传成功
            upSuccess (file) {
                this.$message({
                    message: '上传成功！',
                    type: 'success'
                });
                this.fileList.push({
                    "name": file.resultdata.name,
                    "url": file.resultdata.url,
                });
                this.form.content = this.form.content + "<img src='" + file.resultdata.url +"'>";
            },

            //上传失败
            upError (err, file, fileList) {
                this.$message({
                    message: '上传失败！',
                    type: 'success'
                });
            }
        },
        created () {
            //路由过来的是编辑
            if(this.$route.query.id){
                this.$http.post("/compileArticle",{
                    "id": this.$route.query.id
                },(data)=>{
                    this.form.date = data.time;
                    this.form.title = data.title;
                    this.form.type = data.type;
                    this.form.content = data.content;
                })
            }
        }
    }
</script>
<style lang="less" scoped>
    .ui-title{
        margin-bottom: 20px;
        .ui-h1{
            width: 500px;
        }
        position: absolute;
    }
    .editor-btn{
        margin-top: 20px;
    }
    .upload-demo{
        margin-top: 30px;
    }
</style>
