<template>
  <div id="app">
    <div class="loader fixed top-0 left-0 w-screen h-screen bg-gray-600 opacity-75 flex justify-center items-center" :class="{'hidden': !loading }">Yükleniyor...</div>
    <div class="flex flex-row justify-between bg-pink-600 p-1">
      <select name="markalar" id="markalar" @change="araba_sec">
        <option value="-1">Marka Seç</option>
        <option v-for="araba in markalar" :key="araba.marka" :value="araba.link">{{ araba.marka }} {{ araba.adet }}</option>
      </select>
      <select name="modeller" id="modeller" @change="model_sec">
        <option value="-1">Model Seç</option>
        <option v-for="model in modeller" :key="model.model" :value="model.link">{{ model.model }} {{ model.adet }}</option>
      </select>
      <input class="rounded" type="number" name="min_yil" id="min_yil" v-model="min_yil">
      <input class="rounded" type="number" name="max_yil" id="max_yil" v-model="max_yil">
      <select name="kimden" id="kimden" v-model="kimden">
        <option value="/sahibinden">Sahibinden</option>
        <option value="/galeriden">Galeriden</option>
        <option value="/yetkili-bayiden">Yetkili Bayiden</option>
      </select>
      <button class="bg-white w-32 rounded" @click="getir()">Ara</button>

    </div>
    <div class="flex flex-row w-full bg-blue-600 pt-1 justify-between">
      <button class="bg-green-600 p-3 mb-1 mx-1 w-20 rounded" v-for="page in sayfalar" :key="page.sayfa" @click="getir('https://www.sahibinden.com'+page.link)" v-show="page.sayfa != '...'">{{ page.sayfa }}</button>
    </div>
    <div class="flex flex-col w-full bg-red-600 px-5 pt-5">
      <div class="card w-full h-48 bg-blue-500 rounded-lg flex flex-row mb-5" v-for="araba in arabalar" :key='araba.link'>
        <div class="w-1/5 overflow-hidden">
          <img :src="araba.img" class="h-full w-auto object-cover" alt="">
        </div>
        <div class="flex flex-col w-3/5 border-r-2 border-red-600 p-3 leading-loose">
          <div>
            <a target="_blank" :href="'https://www.sahibinden.com'+araba.link">{{ araba.marka }} {{ araba.model }}</a>
          </div>
          <div class="flex flex-row justify-around">
            <span class="flex-1">Yıl: {{ araba.yil }}</span>
            <span class="flex-1">Km: {{ araba.km }}</span>
          </div>
          <div class="flex flex-row justify-around">
            <span class="flex-1">İl: {{ araba.il }}</span>
            <span class="flex-1">İlçe: {{ araba.ilce }}</span>
          </div>
          <div>
            Tip: {{ araba.tip }}
          </div>
        </div>
        <div class="w-1/5">
          <div class="flex flex-col h-full justify-center items-center p-3 leading-loose">
            <div>
              Fiyat
            </div>
            <div>
              {{ araba.fiyat }} TL
            </div>
            <a target="_blank" :href="'https://www.sahibinden.com'+araba.link">
              <button class="bg-white w-32 rounded">Göster</button>
            </a>
          </div>
        </div>
      </div>
    </div>
    <router-view/>
  </div>
</template>

<script>
export default {
  data() {
    return {
      markalar: [],
      secilen_marka: -1,
      modeller: [],
      secilen_model: -1,
      min_yil: 2010,
      max_yil: 2019,
      kimden: '/sahibinden',
      arabalar: [],
      sayfalar: [],
      loading: true
    }
  },
  methods: {
    araba_sec(e) {
      this.secilen_marka = e.target.value;
      this.marka_getir();
    },
    marka_getir() {
      if(this.secilen_marka != -1) {
        this.loading = true;
        fetch('http://localhost:8888/getModeller', {
          method: 'POST',
          headers: {
              'Content-Type': 'application/json',
              'Accept': 'application/json'
          },
          body: JSON.stringify({
            secilen: this.secilen_marka
          })
        }).then(r => r.json())
        .then(res => {            
            this.modeller = res;
            this.loading = false;
        });
      }
    },
    model_sec(e) {
      this.secilen_model = e.target.value;
    },
    getir(url = undefined) {
      if(this.secilen_marka != -1 && this.secilen_model != -1) {
        this.loading = true;
        if(url == undefined) url = this.getir_url;
        fetch('http://localhost:8888/getir', {
          method: 'POST',
          headers: {
              'Content-Type': 'application/json',
              'Accept': 'application/json'
          },
          body: JSON.stringify({
            url: url
          })
        }).then(r => r.json())
        .then(res => {            
            this.arabalar = res.arabalar.map(x => {
              x.marka = this.markalar.find(k => k.link == this.secilen_marka).marka;
              x.model = this.modeller.find(k => k.link == this.secilen_model).model;
              return x;
            });
            this.sayfalar = res.pages;
            this.loading = false;
        });
      }
    }
  },
  computed: {
    getir_url() {
      return 'https://www.sahibinden.com' + this.secilen_model + this.kimden + '?a5_min=' + this.min_yil + '&a5_max=' + this.max_yil + '&sorting=price_asc';
    }
  },
  created() {
    this.loading = true;
    fetch('http://localhost:8888/getMarkalar', {
        method: 'GET',
        headers: {
            'Content-Type': 'application/json',
            'Accept': 'application/json'
        },
      }).then(r => r.json())
    .then(res => {            
        this.markalar = res;
        this.loading = false;
    });
  }
}
</script>

<style>

</style>