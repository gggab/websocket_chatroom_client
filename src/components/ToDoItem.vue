<template>
  <div class="connection-buttons">
    <button @click="connection">创建连接</button>
    <button @click="close">断开连接</button>
  </div>
  <div class="file-buttons">
    <input type="file" ref="fileInput" @change="onFileInputChange" />
    <button @click="sendFile">发送文件</button>
  </div>
  <div>
    <button @click="getBlendShape">获取音频及表情数据</button>
  </div>
  <div>
    <audio ref="audioPlayer" controls></audio>
  </div>
  <div>
    jawOpen:{{blendShape}}
  </div>
</template>

<script>
import uniqueID from "lodash.uniqueid";

export default {
  props: {
    label: { required: true, type: String },
    done: { default: false, type: Boolean },
    userName: { default: "lwt", type: String },
    nickname: { default: "1421750225", type: String },
  },
  data() {
    return {
      isDone: this.done,
      id: uniqueID("todo-"),
      ws: null,
      selectedFile: null,
      audioSrc: null,
      blendShape:null,
    };
  },
  watch: {
    // audioSrc(newSrc) {
    //   this.$refs.audioPlayer.src = newSrc;
    //   console.log(newSrc)
    //   this.$refs.audioPlayer.play();
    // },
  },
  methods: {
    connection() {
      console.log("connect");
      this.ws = new WebSocket("ws://localhost:3000");
      console.log("before open", this.ws.readyState);

      this.ws.onopen = () => {
        console.log("onopen", this.ws.readyState);
        //   var roomOpen = true;
        // setInterval(() => {
        // this.ws.send(
        //   JSON.stringify({
        //     useId: "lwt",
        //     userName: "1421750225",
        //   })
        // );
        // }, 2000);
      };

      this.ws.onmessage = (message) => {
        // console.log("The client receives the message:", message.data);
        if (message.data !== "建立连接!") {
          const data = JSON.parse(message.data);
          const blendShape = JSON.parse(data.blendShape);
          // const audio = data.audio;
          console.log(blendShape[0]);
          let audioBlob = this.base64ToBlob(data.audio, "wav");
          // console.log(data.audio);

          this.audioSrc = URL.createObjectURL(audioBlob);
          this.$refs.audioPlayer.src = this.audioSrc;
          // console.log(this.audioSrc)
          this.$refs.audioPlayer.play();
          let num=0;
          const oldTime = new Date().getTime();
          let ratio=0;
          let timer = setInterval(() => {
            if(num>11000){
              clearInterval(timer)
              return
            }
            num = new Date().getTime() - oldTime;
            ratio = num/11000;
            const frameCount = parseInt(ratio * blendShape.length);

            const frame = blendShape[frameCount];
            if(frame){
              const jawOpen = frame[73];
              console.log(jawOpen)
            }
          }, 1);
        }
      };

      this.ws.onclose = () => {
        console.log("onclose", this.ws.readyState);
      };
    },
    base64ToBlob(base64, fileType) {
      let typeHeader = "data:application/" + fileType + ";base64,"; // 定义base64 头部文件类型
      let audioSrc = typeHeader + base64; // 拼接最终的base64
      let arr = audioSrc.split(",");
      let array = arr[0].match(/:(.*?);/);
      let mime =
        (array && array.length > 1 ? array[1] : "audio/wav") || "audio/wav";
      // 去掉url的头，并转化为byte
      let bytes = window.atob(arr[1]);
      // 处理异常,将ascii码小于0的转换为大于0
      let ab = new ArrayBuffer(bytes.length);
      // 生成视图（直接针对内存）：8位无符号整数，长度1个字节
      let ia = new Uint8Array(ab);
      for (let i = 0; i < bytes.length; i++) {
        ia[i] = bytes.charCodeAt(i);
      }
      return new Blob([ab], {
        type: mime,
      });
    },
    close: function () {
      console.log("close");
      this.ws && this.ws.close();
    },
    onFileInputChange() {
      const fileInput = this.$refs.fileInput;
      if (fileInput.files.length > 0) {
        this.selectedFile = fileInput.files[0];
      }
    },
    getBlendShape(){
      const fileMetadata = {
        type: "wav",
        name: "video",
      };
      const message = {
        metadata: fileMetadata,
      };
      this.sendData(message);
    },
    sendFile() {
      if (this.selectedFile) {
        const reader = new FileReader();
        reader.onload = () => {
          const fileData = reader.result;
          const fileMetadata = {
            type: "wav",
            name: this.selectedFile.name,
          };
          const message = {
            metadata: fileMetadata,
            data: fileData,
          };
          this.sendData(message);
        };
        reader.readAsArrayBuffer(this.selectedFile);
      }
    },
    sendData(data) {
      if (this.ws) {
        this.ws.send(JSON.stringify(data));
        console.log("文件已发送");
      }
    },
  },
};
</script>
<style>
.connection-buttons,
.file-buttons {
  display: flex;
  justify-content: center;
  align-items: center;
}
button {
  padding: 10px 20px;
  margin: 5px;
}
</style>
