<template>
  <div id="app">
    <h1>Athletic Participation Admin</h1>
    <Submission v-for="item in submissions" :key="item.id" :item="item" @remove="removeItem" />
  </div>
</template>

<script>
import Vue from "vue";
import firebase from "firebase/app";
import "firebase/firestore";
import Submission from "@/components/Submission";

const app = firebase.initializeApp({
  apiKey: "AIzaSyCg_FVHdzP3kvRAyrnpE2bJUsQfMRUgFW4",
  authDomain: "my-covenant.firebaseapp.com",
  databaseURL: "https://my-covenant.firebaseio.com",
  projectId: "my-covenant",
  storageBucket: "my-covenant.appspot.com",
  messagingSenderId: "945207168321",
  appId: "1:945207168321:web:e42d0845df84c8c24e65c0"
});
const store = app.firestore();

export default {
  name: "App",
  components: {
    Submission
  },
  created() {
    store
      .collection("athletic_participation")
      .orderBy("started", "desc")
      .onSnapshot(snapshot => {
        snapshot.docChanges().forEach(change => {
          if (change.type === "added") {
            this.submissions.push({ id: change.doc.id, ...change.doc.data() });
          }
          if (change.type === "modified") {
            const index = this.submissions.findIndex(i => {
              return i.id === change.doc.id;
            });
            Vue.set(this.submissions, index, {
              id: change.doc.id,
              ...change.doc.data()
            });
          }
          if (change.type === "removed") {
            this.submissions = this.submissions.filter(
              i => i.id !== change.doc.id
            );
          }
        });
      });
  },
  data: () => {
    return {
      submissions: []
    };
  },
  methods: {
    removeItem(id) {
      store.doc(`athletic_participation/${id}`).delete();
    }
  }
};
</script>

<style>
@import url("https://fonts.googleapis.com/css2?family=Work+Sans:wght@400;700&display=swap");

html,
body {
  font-family: "Work Sans";
}
</style>
