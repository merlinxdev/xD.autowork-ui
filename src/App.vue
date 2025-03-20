<template>
  <div v-if="main_box" :class="{
    ['main-box']: true,
    ['hide-main-box']: Animation.fade_out
  }">
    <div class="box-body">
      <div class="box-title">
        <div class="rt1"></div>
        <span>{{ work.work_name }}</span>
        <div class="rt2"></div>
      </div>
      <div class="flex">
        <div class="box-image" :style="getPercenPickup()">
          <div class="circle"></div>
          <img :src="getImage(work.itemName)" alt="">
        </div>
        <div class="box-count">
          <span>{{ work.pickupCount }}</span>
          <div class="line1"></div>
          <span style="color: red;">{{ work.maxItem }}</span>
        </div>
      </div>
      <div class="box-bonus">
        <div class="box-detail-bonus" v-for="(item,index) in bonus" :key="index">
          <div class="box-img-bonus">
            <img :src="getImage(item.itemName)" alt="">
            <span>{{ item.itemCount }}</span>
          </div>
        </div>
      </div>
      <div class="space"></div>
      <div class="box-footer">กด <btn>X</btn> เพื่อยกเลิก</div>
    </div>
  </div>
</template>

<script>

export default {
  name: 'App',
  data() {
    return {
      progress: 0, // เปอร์เซ็นต์ปัจจุบัน
      Animation :{
        fade_out : false,
      },
      main_box: false,
      bonus_box: false,
      itemLocation : ``,
      work : {
          work_name : 'งานเก็บแตงโม',
          itemName : 'water_melon.png',
          maxItem : 0,
          pickupCount : 0
      },
      bonus : [
        {
          itemName : 'water',
          itemCount : 2,
        }
      ]
    }
  },
  computed: {

  },
  methods: {
    sendLuaFetch(type, data) {
      let requestOptions = {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(data)
      };
      fetch(`https://xD.autowork/${type}`, requestOptions)
    },
    getImage(itemName){
      const img = `${this.itemLocation}${itemName}.png`
      return img;
    },
    startPickup(time) {
      const startTime = Date.now();
      const endTime = startTime + time;
      
      this.interval = setInterval(() => {
        const now = Date.now();
        const elapsedTime = now - startTime;
        const percentage = Math.min((elapsedTime / time) * 100, 100);

        this.progress = percentage;

        if (now >= endTime) {
          clearInterval(this.interval);
        }
      }, 16);
    },
    getPercenPickup(time){
        return `background: conic-gradient(rgb(219, 15, 15) ${this.progress}%, #e0e0e0 0deg);`
    },
    openUI(){
      this.main_box = true
      this.Animation.fade_out = false
    },
    closeUI(){
        this.Animation.fade_out = true
        setTimeout(() => {
          this.main_box = false
        }, 1000);
    },
    async onMessage(event) {
      let data = event.data
      if (data.type == 'XD_NUI_SETUP') {
        this.itemLocation = data.data.imgpart
      }
      if (data.type == 'XD_NUI_SHOW') {
        if(data.data.ui){
          this.openUI()
        }else{
          this.closeUI()
        }
      }
      if (data.type == 'XD_NUI_PICKUP') {
        this.startPickup(data.data.time);
      }
      if (data.type == 'XD_NUI_UPDATE_ฺBONUS') {
        this.bonus = data.data.bonus
      }
      if (data.type == 'XD_NUI_UPDATE_INFO') {
        this.work.maxItem = data.data.itemMax
        this.work.work_name = data.data.workName
        this.work.itemName = data.data.itemName
        this.work.pickupCount = data.data.itemCount
        this.bonus = data.data.bonus
      }

      if (data.type == 'XD_NUI_UPDATE_COUNT') {
          this.work.pickupCount = data.data.itemCount
      }

    },
    onKeyup(event) {
      if (event.keyCode == 27) {
        this.work.maxItem = 22
      }
    }
  },
  mounted() {
    window.addEventListener('message', this.onMessage)
    window.addEventListener('keyup', this.onKeyup)
  },
}

</script>