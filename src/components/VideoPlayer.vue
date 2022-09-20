<template>

   <div class="vertical-flex" v-if="done">
      <span>Video Player</span>
      <video preload="auto" ref="video1El" v-show="!altVideoActive" controls :src="srcs[0]"
         @ended="clipEnded($event, 0)" @load="test($event)"></video>
      <video preload="auto" ref="video2El" v-show="altVideoActive" controls :src="srcs[1]" @ended="clipEnded($event, 1)"
         @loadeddata="test($event)"></video>
   </div>

</template>

<script lang="ts">

import { defineComponent, reactive, ref } from 'vue'

//probably going to have to create a full video on a backend and send it to the FE
//maybe not - current problem is that there's a "flicker" in between clips. looking into putting 2 video players on the screen
//so the following clip can be preloaded while the 2nd player is hidden

export default defineComponent({
   name: "video",
   setup() {
      let done = false;

      // function importAll(r: any) {
      //    return r.keys().map(r);
      // }
      const videoElements = [] as HTMLVideoElement[];
      const filenames = import.meta.glob("/src/assets/saul randomizer clips/*.mp4");
      let videoNum = 1;
      for (const x in filenames) {
         console.log(filenames[x].name)
         const video = document.createElement(`video${videoNum}`) as HTMLVideoElement
         video.src = filenames[x].name;
         videoElements.push(video)
      }

      // const videoElementsRef = reactive([] as HTMLVideoElement[]);
      const srcs = reactive([] as string[])

      function shuffleStartingArray(array: HTMLVideoElement[]) {
         //Randomize array in-place using Durstenfeld shuffle algorithm
         for (let i = array.length - 1; i > 0; i--) {
            const j = Math.floor(Math.random() * (i + 1));
            [array[i], array[j]] = [array[j], array[i]];
         }
      }
      shuffleStartingArray(videoElements)

      srcs.push(videoElements[0].src)
      srcs.push(videoElements[1].src)


      let altVideoActive = ref(false);
      let count = 1;
      done = true;
      return {
         done,
         videoElements,
         altVideoActive,
         srcs,
         count
      }
   },

   methods: {

      test(event: any) {
         console.log(event)
      },
      clipEnded(event: any, index: number) {
         console.log("clip ended")
         this.count++;
         console.log("count " + this.count)
         console.log("videos")
         console.log(this.videoElements)
         if (this.count >= this.videoElements.length) this.count = 0;
         if (index === 0) {
            (this.$refs["video2El"] as HTMLVideoElement).controls = false;
            (this.$refs["video2El"] as HTMLVideoElement).play();
            this.altVideoActive = true;


            this.videoElements.splice(0, 1); //cut ended clip out of source array
            //this.srcs.splice(0, 1, this.videoElements[0].src)
            if (this.videoElements.length < this.count - 1) return;
            this.srcs[0] = this.videoElements[this.count].src

         }
         else {
            (this.$refs["video1El"] as HTMLVideoElement).controls = false;
            (this.$refs["video1El"] as HTMLVideoElement).play();
            this.altVideoActive = false;


            this.videoElements.splice(0, 1); //cut ended clip out of source array
            //this.srcs.splice(1, 1, this.videoElements[1].src)
            if (this.videoElements.length < this.count - 1) return;
            this.srcs[1] = this.videoElements[this.count].src

         }

      },




   }



})

</script>

<style scoped>
.hidden {
   position: absolute;
   left: -100vw;
}
</style>