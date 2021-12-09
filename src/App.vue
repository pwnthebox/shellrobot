<template>
  LINUX SHELL 机器人<br><br>
  运行状态：{{status ? '运行中': '未运行'}}<br><br>
  <button @click="swit">{{status ? '停止': '启动'}}</button>
</template>

<script>
import 'mirai-js'
const { Bot, Message } = window.miraiJs

export default {
  name: 'App',
  data(){
    return {
      status: false,
      bot: undefined,
      data: '',
      term: undefined
    }
  },
  mounted(){
    this.bot = new Bot()
    this.bot.open({
        baseUrl: 'http://localhost:1234',
        verifyKey: 'AAAAAAAAAAAA',
        qq: 'QQ',
    });
    this.bot.on('GroupMessage', async data => {
        data.messageChain.map(e=>{
            let message = e.text
            if(message)message = message.replace(/[^\x20-\x7E]/g,'');
            if (e.type == 'Plain' && message.search(/^(echo).*/) == 0){
              this.term.serial0_send(message + '\n');
            }
            if (e.type == 'Plain' && message == "reset"){
              localStorage.setItem('autoStart', 'true')
              location.reload()
            }
        })
    });
    if(localStorage.getItem('autoStart') == 'true'){
      this.swit()
    }
    this.$nextTick(() => {
      setInterval(this.CurentTime, 3000);
    })
  },
  methods:{
    swit(){
      const that = this
      if(this.status){
        localStorage.clear()
        location.reload()
      }else{
        this.term = new window.V86Starter({
            memory_size: 32 * 1024 * 1024,
            bios: {
              url: './seabios.bin',
            },
            bzimage: {
              url: './buildroot.bin',
              size: 5166352,
              async: false,
            },
            autostart: true
        });
        this.term.add_listener("serial0-output-char", function(chr)
        {
          that.data += chr
        })
        this.status = !this.status
      }
    },
    CurentTime(){
      let passdata = this.data
      console.log(passdata)
      if(passdata.substr(passdata.length-2, 2) == "% " && passdata.indexOf('\n') !== -1){
        if(passdata.indexOf("Enable networking using 'udhcpc'") !== -1){
            setTimeout(()=>{
                this.term.serial0_send('echo "flag{CTF_QqUn_7072_126}" > /flag\n');
                this.bot.sendMessage({
                    group: '群号',
                    message: new Message().addText("群靶机启动成功！只允许使用 echo 指令。"),
                });
            }, 5000);
        }else{
          // eslint-disable-next-line
          const _data = passdata.replace(/[\u001b\u009b][[()#;?]*(?:[0-9]{1,4}(?:;[0-9]{0,4})*)?[0-9A-ORZcf-nqry=><]/g, '')
          if(_data.trim() !== ""){
              this.bot.sendMessage({
                  group: '群号',
                  message: new Message().addText(_data.slice(0, _data.lastIndexOf('\n'))),
              });
          }
        }
        this.data = this.data.slice(passdata.lastIndexOf('\n') + 1)
      }
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
