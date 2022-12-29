
<template>
  <div class="w-full primary-bg min-h-screen">
         <div class="container h-42 shadow py-10 px-6 bg-white">
            <div class="flex items-center justify-between">
              <div>
                <select @change="getSpecificSurah" class="py-2 px-4 border rounded border-gray-500 focus:outline-none" name="">
                  <option>Select Sura</option>
                  <option  v-for="sura in shuras" :value="sura.number" :key="sura.id">{{sura.name}} - {{sura.englishName}}</option>
                </select>
              </div>

              <div>
                <h3>{{sura.englishName}}</h3>
                <p>{{sura.englishNameTranslation}}</p>
              </div>
              <div>
                <select @change="fetchedSpecificAyahs" v-model="isSelected" class="py-2 px-4 border rounded border-gray-500 focus:outline-none">
                  <option value="reset">Select Ayahs</option>
                  <option v-for="ayahs in ayahss" :key="ayahs.id" :value="ayahs.number">Ayahs No-{{ayahs.numberInSurah}}</option>
                </select>
              </div>
            </div>
           <div  class="flex mt-6 flex-wrap w-full">
             <div class="w-full" v-if="ayahs.hasOwnProperty('arbicText')">
               <p class="primary-color text-xl py-4 leading-10"> <span class="border-rose-900 border-2 text-indigo-900 w-6 h-6 inline-block text-center border rounded-full mx-2 text-sm">{{ayahs.numberInSurah}} </span>{{ayahs.arbicText}}</p>
               <p class="primary-color text-xl py-4 leading-10">=> {{ayahs.banglaText}}</p>
               <p class="primary-color text-xl py-4 leading-10">=> {{ayahs.enlishText}}</p>
               <div class="w-[400px] mx-auto">
                 <audio-player
                     ref="audioPlayer"
                     :audio-list="audios"
                     :isLoop="true"
                     :before-play="handleBeforePlay"
                     theme-color="#ff2929"
                     :is-auto-play-next="true"
                     :before-next="contentChange"
                 />
               </div>
             </div>

             <p v-else class="primary-color text-xl py-4 leading-10" v-if="sura.hasOwnProperty('ayahs')" v-for="ayahs in sura.ayahs" :key="ayahs.id"> <span class="border-rose-900 text-indigo-900 w-6 h-6 inline-block text-center border rounded-full mx-2 text-sm">{{ayahs.numberInSurah}} </span>{{ayahs.text}}</p>
           </div>
         </div>
  </div>
</template>

<script>
import axios from "axios";
import AudioPlayer from '@liripeng/vue-audio-player'

export default {
  name: 'Quran App',
  components: {
    AudioPlayer
  },
  data () {
    return {
      isSelected: 'reset',
      shuras: [],
      sura:{},
      ayahs:{},
      ayahss: [],
      ayahsNumber:0,
      ayahsNumber2:0,
      audios:[],

    }
  },
  mounted() {
    axios.get('https://api.alquran.cloud/v1/surah').then((response) => {
      this.shuras = response.data.data;
    });
   this. fetchSpecificSurah(1);
  },
  methods: {
    handleBeforePlay(next) {
      let aynum = this.ayahsNumber + 1;
      let highNum = this.ayahss[this.ayahss.length - 1].number;
      if (aynum <=highNum) {
        axios.get(`https://api.alquran.cloud/v1/ayah/${aynum}/editions/ar.alafasy,bn.bengali,en.asad`).then((response) => {
          this.audios = [...this.audios,response.data.data[0].audio]
          this.ayahsNumber = response.data.data[0].number;
        });
      }
      next();
    },
    contentChange(next){
      let aynum = this.ayahsNumber2 + 1;
      let highNum = this.ayahss[this.ayahss.length - 1].number;
      if (aynum <=highNum) {
        console.log('okk1')
      axios.get(`https://api.alquran.cloud/v1/ayah/${aynum}/editions/ar.alafasy,bn.bengali,en.asad`).then((response) =>{
        this.ayahs = {arbicText: response.data.data[0].text, banglaText: response.data.data[1].text, numberInSurah: response.data.data[0].numberInSurah, enlishText: response.data.data[2].text, audio: response.data.data[0].audio };
        this.isSelected = response.data.data[0].number
        this.ayahsNumber2 = response.data.data[0].number;
       })
      }else{
        return false;
      }

      next();
    },

    getSpecificSurah(e){
      return this.fetchSpecificSurah(e.target.value);
    },
    fetchSpecificSurah(id){
      axios.get(`https://api.alquran.cloud/v1/surah/${id}`).then((response) =>{
        this.sura = response.data.data;
        this.ayahss = response.data.data.ayahs;
        this.ayahs = {};
        this.isSelected = 'reset'
      });
    },

    fetchedSpecificAyahs(e){
      axios.get(`https://api.alquran.cloud/v1/ayah/${e.target.value}/editions/ar.alafasy,bn.bengali,en.asad`).then((response) =>{
        this.ayahs = {arbicText: response.data.data[0].text, banglaText: response.data.data[1].text, numberInSurah: response.data.data[0].numberInSurah, enlishText: response.data.data[2].text, audio: response.data.data[0].audio };
        this.audios = [response.data.data[0].audio]
        this.ayahsNumber = response.data.data[0].number;
        this.ayahsNumber2 = response.data.data[0].number;

      })
    }

  }
}
</script>
