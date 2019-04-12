<template>
    <div class="home" ref="home">
        <button v-if="!text.length" class="btn" :class="[isDraging && 'hover']" @click="openFile">上传文件</button>
        <div v-else class="text-wrap">
            <div class="content">
                <p v-for="(t, index) in text" :key="t" @click="nextLine(index, $event)">{{ t }}</p>
            </div>
            <div class="operation">
                <span @click="back" v-show="backup.length">撤退</span>
            </div>
        </div>  
    </div>
</template>
<script>
const fs = require('fs')
const path = require('path')
export default {
    name: 'home',
    data(){
        return {
            isDraging: false,
            text: [],
            backup: []
        }
    },
    mounted(){
        document.addEventListener('drop', this.dropHandle);
        this.$refs.home.addEventListener('dragover', this.dragoverhandle);
    },
    beforeDestory(){
        document.removeEventListener('drop', this.dropHandle)
        document.removeEventListener('dragover', this.dragoverhandle)
    },
    methods:{
        readFile(path1){
            fs.readFile(path1, 'utf8', (err, text) => {
                if(err) return console.log('read file error')
                this.transferText(text)
            })
        },
        openFile(){
            this.$electron.remote.dialog.showOpenDialog({ properties: ['openFile'], title: '选择文件' }, (filePaths) => {
                this.readFile(filePaths[0])
            })
        },
        dropHandle(e){
            e.preventDefault();
            e.stopPropagation();
            this.isDraging = false
            this.fileToText(e.dataTransfer.files[0])
        },
        dragoverhandle(e){
            this.isDraging = true
            e.preventDefault();
            e.stopPropagation();
        },
        fileToText(file){
            const reader = new FileReader();
            reader.readAsText(file)
            reader.onload = (e) => {
                this.transferText(e.target.result)
            }
        },
        transferText(text){
            this.text = text.split(/[,，。]/)
        },
        nextLine(index, e){
            var FontSize = 16;
            var text = this.text[index]
            var textIndex = Math.floor(e.offsetX / FontSize) + 1
            if(FontSize*text.length < e.offsetX) return 
            this.backup.push(JSON.parse(JSON.stringify(this.text)))
            if(textIndex > 0){
                this.text.splice(index, 1, text.slice(0, textIndex), text.slice(textIndex))
            }
        },
        back(){
            let text = this.backup.pop()
            if(text){
                this.text = text
            }
        }
    }
}
</script>
<style scoped>
.home{
    min-height: 100%;display: flex; justify-content: center; align-items: center; flex-direction: column; 
}
.btn{
    padding: 10px 20px; min-width: 100px;text-align: center; background: #009fff; color: #fff; font-size: 16px;
    transition: all .5s;
}
.btn:hover,.btn.hover{
    background: #007fff; cursor: pointer;
}
.text-wrap{
    width: 80%;
}
.content{
    padding: 40px; line-height: 1.7; font-size: 16px; color: #444;
}
.content p{
    margin-bottom: 15px; 
}
.operation{
    height: 60px; width: 100%; position: fixed; bottom: 0; left: 0;
    z-index: 10; color: #007fff; 
    display: flex; align-items: center; justify-content: space-around;
}
.operation span{
    cursor: pointer;
}
</style>


