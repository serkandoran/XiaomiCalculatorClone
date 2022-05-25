<template>
  <div id="calc_container">
    <div id="top_container">
      <div id="top" >
        <span id="pastDataSpan" v-for="(result, index) in passiveData" :key="index">
          {{result}}
        </span>
        <span id="res">{{res==''?0:res}}</span>
        <span id="passiveSpan">{{res=='' || res==null ?instantRes='':`=${instantRes}`}}</span>
      </div>
    </div>
    <hr>
    <div id="bottom">
      <div @click="ac()" id="ac">AC</div>
      <div @click="del()" id="del">Del</div>
      <div @click="percent()" id="percent">%</div>
      <div @click="operandF('/')" id="divide">/</div>

      <div @click="numberF('7')" id="seven">7</div>
      <div @click="numberF('8')" id="eight">8</div>
      <div @click="numberF('9')" id="nine">9</div>
      <div @click="operandF('x')" id="multiple">x</div>

      <div @click="numberF('4')" id="four">4</div>
      <div @click="numberF('5')" id="five">5</div>
      <div @click="numberF('6')" id="six">6</div>
      <div @click="operandF('-')" id="minus">-</div>

      <div @click="numberF('1')" id="one">1</div>
      <div @click="numberF('2')" id="two">2</div>
      <div @click="numberF('3')" id="three">3</div>
      <div @click="operandF('+')" id="plus">+</div>

      <div id="expand">e</div>
      <div @click="numberF('0')" id="zero">0</div>
      <div @click="comma(',')" id="comma">,</div>
      <div @click="equalF()" id="equals"><span>=</span></div>
    </div>
  </div>
</template>

<script>
export default {
  data(){
    return{
      res:'',
      operand:null,
      instantRes:'',
      operandCanCome:true,
      commaCanCome:true,
      pressedEqual:false,

      operandSymbols:['x','/','-','+'],
      numberSymbols:['0','1','2','3','4','5','6','7','8','9'],

      passiveData:[]
    }
  },
  methods:{
    numberF(val){
      if(this.pressedEqual == true){
        this.passiveData.push(this.res,'='+this.instantRes)
        this.res=''
        this.instantRes = ''
        document.getElementById('res').classList.remove('res')
        document.getElementById('passiveSpan').classList.remove('passiveSpan')
        this.pressedEqual = false
      }
      if(this.operand == null){
        this.instantRes=this.instantRes+val
      }
      if(this.operand != null || this.commaCanCome==false){
        this.res+=val
        let fakeVal = this.res
        if(this.res.includes('x')){
          let find = 'x'
          let rgx = new RegExp(find,'g')
          fakeVal = fakeVal.replace(rgx,'*')
        }
        if(this.res.includes(',')){
            let find = ','
            let rgx = new RegExp(find,'g')
            fakeVal = fakeVal.replace(rgx,'.')
        }
        this.instantRes = Function(
          `return ${fakeVal}`
        )()
      }else this.res+=val
      this.operandCanCome = true
    },
    operandF(op){
      if(this.pressedEqual == true){
        this.passiveData.push(this.res,'='+this.instantRes)
        this.res = this.instantRes
        this.operandCanCome=true
        document.getElementById('res').classList.remove('res')
        document.getElementById('passiveSpan').classList.remove('passiveSpan')
        if(this.res!='' && this.operandCanCome){
          this.res+=op
          this.operand = op
          this.operandCanCome = false
          this.commaCanCome=true
        }
        this.pressedEqual = false
      }else{
        if(this.operandSymbols.includes(this.res[this.res.length-1])){
          if(this.operandCanCome == false) this.res = this.res.substring(0,this.res.length-1)+op
        }
        if(this.res!='' && this.operandCanCome){
          this.res+=op
          this.operand = op
          this.operandCanCome = false
          this.commaCanCome=true
        }
      }
    },
    hesapla(param){
      return Function(`return ${param}`)()
    },
    regDegistir(str){
      if(str.includes(',')){
        let find = ','
        let reg = new RegExp(find,'g')
        str = str.replace(reg,'.')
      }
      if(str.includes('x')){
        let find = 'x'
        let reg = new RegExp(find,'g')
        str = str.replace(reg,'*')
      }
      return str
    },
    del(){
      if(this.res.length > 0 && this.pressedEqual==false){
        let fakeVal = null
        
        if(this.operandSymbols.includes(this.res[this.res.length-1])){
          this.res = this.res.substring(0,this.res.length-1)
          this.operandCanCome = true
        }else if(this.res[this.res.length-1]==','){
          this.res = this.res.substring(0,this.res.length-1)
          this.commaCanCome = true
        }else{
          this.res = this.res.substring(0,this.res.length-1)
        }
        fakeVal = this.res
        if(this.operandSymbols.includes(fakeVal[fakeVal.length-1])){
          fakeVal = fakeVal.substring(0,fakeVal.length-1)
        }
        fakeVal = this.regDegistir(fakeVal)
        this.instantRes = this.hesapla(fakeVal)


      }
    },
    comma(op){
      if(this.res==''){
        this.res='0'+op
        this.commaCanCome = false
      }else{
        if(this.commaCanCome){
          this.res+=op
          this.commaCanCome = false
        }
      }


    },
    percent(){
      if(!this.operandSymbols.includes(this.res[this.res.length-1]) && this.res!=''){
        let percentComes = ''
        let left = ''
        for(let i=this.res.length-1; i>=0; i--){
          if(!this.operandSymbols.includes(this.res[i])){
            percentComes+=this.res[i]
          }
          if(this.operandSymbols.includes(this.res[i])){
            break
          }
        }
        percentComes = percentComes.split('').reverse().join('')
        for(let i=0; i<this.res.length-percentComes.length; i++){
          left+=this.res[i]
        }
        percentComes = this.regDegistir(percentComes)
        percentComes = this.hesapla(percentComes)/100
        this.instantRes = this.regDegistir(left+percentComes)
        this.instantRes = this.hesapla(this.instantRes)
  
        percentComes = String(percentComes)
        this.res = left+percentComes
        for(let i=0; i<this.res.length; i++){
          if(this.res[i]=='.'){
            this.res = this.res.substring(0,this.res.indexOf('.'))+','+this.res.substring(this.res.indexOf('.')+1)
          }
        }
      }

    },
    equalF(){
      if(this.res!=''){
        this.pressedEqual = true
        document.getElementById('res').classList.add('res')
        document.getElementById('passiveSpan').classList.add('passiveSpan')
      }
    },
    ac(){
      if(this.res!=''){
        this.res = ''
        this.instantRes = ''
        document.getElementById('res').classList.remove('res')
        document.getElementById('passiveSpan').classList.remove('passiveSpan')
      }else if(this.res==''&&this.passiveData.length>0){
        this.passiveData = []
      }
    }
  },
  watch:{
    res:{
      handler(val){
        if(val!=''){
          document.getElementById('ac').innerHTML = 'C'
        }
      }
    },
    passiveData:{
      handler(val){
        if(val.includes('')){
          this.passiveData=[]
        }
      }
    }
  },
  created(){

  }
}
</script>
<style>
#calc_container>*{
  padding: 0;
  margin: 0;
  color: white;
}
#calc_container{
  background: rgb(14, 13, 13);
  height: 700px;
  width: 375px;
  position: absolute;
  left: 0;
  top: 0;
  right: 0;
  bottom: 0;
  margin: auto;
  display: flex;
  flex-direction: column;
  align-items: center;
}
hr{
  background: rgb(82, 81, 81);
  width: 85%;
  margin: 0 auto;
}
#bottom{
  width: 100%;
  height: 65%;
  display: flex;
  flex-wrap: wrap;
  margin-top: 1.5rem;
}
#bottom > div{
  width: 25%;
  text-align: center;
  cursor: pointer;
  font-size: 20px;
}
#ac,#del,#percent,#divide,#multiple,#minus,#plus{
  color: rgb(255, 123, 0);
  font-weight: bold;
}
#ac,#del,#divide,#percent{
  font-size: 18px !important;
}
#expand{
  color: rgb(255, 123, 0);
}
#equals{
  color: rgb(255, 123, 0);
}
#equals > span{
  background: rgb(255, 123, 0);
  display: block;
  width: 50px;
  height: 40px;
  margin: 0 auto;
  padding: 0;
  border-radius: 150px;
  color: white;
}
#top_container{
  width: 100%;
  height: 35%;
  margin-bottom: 1.5rem;
  display: flex;
  flex-flow: column-reverse;
  overflow: hidden;
}
#res{
  display: block;
  text-align-last: right;
  padding-right: 1.5rem;
  font-size: 2.2rem;
  transition: all .1s;
}
.res{
  font-size: 1.3rem !important;
}
#passiveSpan{
  display: block;
  text-align-last: right;
  padding-right: 1.5rem;
  font-size: 1.3rem;
  color: rgb(129, 129, 129);
  transition: all .1s;
}
.passiveSpan{
  font-size: 2.2rem !important;
  color: white !important;
}
#pastDataSpan{
  display: block;
  text-align-last: right;
  padding-right: 1.5rem;
  font-size: 1rem;
  color: rgb(129, 129, 129);
}
</style>