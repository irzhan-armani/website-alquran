<script setup>
import TheWelcome from '@/components/TheWelcome.vue'
import NavBar from "../components/NavBar.vue";
import { onMounted } from "vue";

onMounted(() => {
  document.getElementsByTagName("body")[0].style.backgroundImage ="Url('src/assets/back.jpg')"
  document.getElementsByTagName("body")[0].style.backgroundSize ="cover";
});
</script>

<template>


  <main>
    <NavBar/>
    <section class="header">
      <div class="display-5 text-center fw-bolder">
        <p class="title">Al-Qur'an ku</p>
        <div class="display-6 text-center fs-6">
          <p class="intro fs-6">Website ini akan merupakan Website yang akan menampilkan bacaan Al-Qur'an, mulai dari info surah, terjemahan hingga audionya.</p>
        </div>
      </div>
    </section>
    <!-- Akhir header-->

    <!-- search-->
    <section class="search">
      <input type="number" v-model="inputnomor" class="input" placeholder="Masukkan urutan surah" />
      <button class="btn btn-primary" @click="submit" type="submit">Cari</button>
    </section>
    <!-- Akhir search-->

    <!-- Utama -->
    <section class="container my-5">
      <div class="row justify-content-center">
        <div class="col-12">
          <div class="card rounded shadow">
            <div class="card-header display-5 text-center fw-bolder">
              <p v-if="namaSurah" class="judul">{{ namaSurah.name_simple }}</p>
              <p v-if="namaSurah" class="judul fs-1">{{ namaSurah.name_arabic }}</p>
              <p v-if="namaSurah" class="judul fs-6 text-start fw-normal">Jumlah Ayat : {{ namaSurah.verses_count }}</p>
              <p v-if="namaSurah" class="judul fs-6 text-start fw-normal">Diturunkan di : {{ namaSurah.revelation_place }}</p>
              <p v-if="namaSurah" class="judul fs-6 text-start fw-normal">Surah ke : {{ namaSurah.revelation_order }} diturunkan</p>
              <p v-if="audio">
                <audio controls class="suara1">
                  <source :src="audio.audio_url" type="audio/mpeg" />
                  Browser Anda tidak mendukung elemen audio.
                </audio>
              </p>
            </div>
            <div class="bismillah"></div>
            <div class="ayat" v-for="ayat in ayah" :key="ayat.id">{{ ayat.text_uthmani }} {{ ayat.text }}</div>
          </div>
        </div>
      </div>
    </section>
    <!-- Akhir Utama -->



  </main>
</template>
<script>
import axios from "axios";

export default {
  data() {
    return {
      ayah: [],
      audio: null,
      namaSurah: null,
      inputnomor: "",
    };
  },

  methods: {
    async submit() {
      let nomor = this.inputnomor;
      let ayat = `https://api.quran.com/api/v4/quran/verses/uthmani?chapter_number=${nomor}`;
      let arti = "https://api.quran.com/api/v4/quran/translations/134?chapter_number=" + nomor;

      let judul = "https://api.quran.com/api/v4/chapters?language=en";
      let suara = "https://api.quran.com/api/v4/chapter_recitations/2?language=en";

      if (nomor <= 0 || nomor > 114) {
        alert("nomor surah yang dimasukkan salah!");
      } else {
        const reqJudul = axios.get(judul);
        const reqAyat = axios.get(ayat);
        const reqSuara = axios.get(suara);
        const reqArti = axios.get(arti);

        axios.all([reqAyat, reqArti, reqJudul, reqSuara]).then(
            axios.spread((...response) => {
              const responseAyat = response[0];
              const responseArti = response[1];
              const responseJudul = response[2];
              const responseSuara = response[3];

              const a = responseAyat.data.verses;
              const b = responseArti.data.translations;

              const gabung = (a, b) => {
                const res = [];

                for (let i = 0; i < a.length + b.length; i++) {
                  if (i % 2 === 0) {
                    res.push(a[i / 2]);
                  } else {
                    res.push(b[(i - 1) / 2]);
                  }
                }
                return res;
              };

              this.ayah = gabung(a, b);
              this.audio = responseSuara.data.audio_files[nomor - 1];
              this.namaSurah = responseJudul.data.chapters[nomor - 1];
            })
        );
      }
    },
  },
};
</script>
<style>
.header {
  display: flex;
  margin: 50px 0 0 0;
  flex-direction: row;
  justify-content: center;
}

.search {
  display: flex;
  margin: 30px 0 0 0;
  flex-direction: row;
  justify-content: center;
}
.title {
  font-size: 60px;
  margin-top: 20px;
  color: #eee8e8;
}
.intro {
  font-size: 30px;
  color: #eee8e8;
}

.input {
  height: 30px;
  border: 1px solid #323232;
  color: #4c4c4c;
}

.input:hover {
  border: 1px solid #6d9581;
}

.btn {
  background-color: #57f0d9;
  border: 1px solid #323232;
  height: 30px;
  font-size: 12px;
  color: black;
  border-radius: 5%;
}

.btn:hover {
  color: wheat;
}

.suara1 {
  width: 70%;
  height: 30px;
}

.suara {
  margin: 50px 0;
  text-align: center;
}

.judul {
  text-align: center;
  font-size: 70px;
  color: #6495ed;
}

.bismillah {
  text-align: center;
  font-size: 50px;
  margin-top: 10px;
  margin-bottom: 40px;
  color: #4c4c4c;
}

.ayat {
  color: #4c4c4c;
  list-style: none;
  margin: 0 100px 30px 100px;
}

.ayat:nth-child(odd) {
  text-align: right;
  font-size: 40px;
}
.ayat:nth-child(even) {
  text-align: left;
  font-size: 15px;
  color: #747474;
}

@media screen and (max-width: 400px) {
  .ayat:nth-child(odd) {
    font-size: 20px;
    margin-bottom: 20px;
    margin-right: 10px;
  }
  .ayat:nth-child(even) {
    font-size: 10px;
    margin: 20px;
  }
  .ayat {
    margin: 0;
  }
}
</style>
