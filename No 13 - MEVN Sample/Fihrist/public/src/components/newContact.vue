<template>
  <div>
    <h1>Yeni Bağlantı</h1>
    <form>
      <div class="form-group">
        <label>Fullname</label>
        <input
          type="text"
          class="form-control"
          aria-describedby="inputGroup-sizing-default"
          placeholder="nasıl isimlendirirsin?"
          v-model="contact.fullname"
        >
      </div>
      <div class="form-group">
        <label>Phone</label>
        <input
          type="text"
          class="form-control"
          aria-describedby="inputGroup-sizing-default"
          placeholder="nereden ulaşırsın?"
          v-model="contact.phoneNumber"
        >
      </div>
      <div class="form-group">
        <label>Location</label>
        <input
          type="text"
          class="form-control"
          aria-describedby="inputGroup-sizing-default"
          placeholder="nerede yaşıyor?"
          v-model="contact.location"
        >
      </div>
      <div class="form-group">
        <label>Birthdate</label>
        <input
          type="text"
          class="form-control"
          aria-describedby="inputGroup-sizing-default"
          placeholder="1976-04-12T11:35:00Z"
          v-model="contact.birtDate"
        >
      </div>
      <div class="form-group">
        <button type="button" class="btn btn-primary" @click="createContact($event)">Kaydet</button>
      </div>
    </form>
  </div>
</template>

<script>
import axios from "axios"; // API servis haberleşmesi için
import bus from "./../bus.js"; // bileşenler arası haberleşme için
// HTML elementlerindeki input kontrollerinde dikkat edileceği üzere v-model attribute'ları kullanıldı. Bunlar modelimizin özellikleri.

export default {
  data() {
    return {
      contact: {
        fullname: "",
        phoneNumber: "",
        location: "",
        birtDate: ""
      }
    };
  },
  methods: {
    createContact(event) {
      //Button'un @click niteliğinde yüklenen olay metodu
      if (event) event.preventDefault();
      let url = "http://localhost:5003/api";
      let param = {
        //parametre değerleri input kontrollerinden geliyor
        fullname: this.contact.fullname,
        phoneNumber: this.contact.phoneNumber,
        location: this.contact.location,
        birtDate: this.contact.birtDate
      };
      axios
        .post(url, param) //HTTP Post çağrısını gönderdik
        .then(response => {
          console.log(response); // tarayıcının developer tool kısmından log takibi için. Canlı ortamda kullanmaya gerek yok.
          this.clear();
          this.refresh(); // yeni bir bağlantı oluşturlduğunda refresh metodu çağırılır
        })
        .catch(error => {
          console.log(error);
        });
    },
    clear() {
      this.contact.fullname = "";
      this.contact.phoneNumber = "";
      this.contact.location = "";
      this.contact.birtDate = "";
    },
    refresh() {
      bus.$emit("refresh"); // metod evenbus'a bir olay fırlatır. refresh isminde. bunu diğer bileşende yakalayarak bağlantılar listesini anında günelleyebiliriz
    }
  }
};
</script>
