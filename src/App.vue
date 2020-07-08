<template>
  <div id="app">
    <h1>Athletic Participation Admin</h1>
    <div v-if="user">
      Show Updated EventLink Submissions
      <input type="checkbox" v-model="showEventLink" />
      &nbsp;&nbsp;
      Total: {{ filteredSubmissions.length }}
      &nbsp;&nbsp;
      Filter by Student:
      <input
        v-model="filter"
      />
      <Submission
        v-for="item in filteredSubmissions"
        :key="item.id"
        :item="item"
        @eventlink="eventLink"
        @ihsaa="updatePage"
        @remove="removeItem"
      />
    </div>
    <div v-else>
      <p>You must login to access this site.</p>
      <button @click="google">Login with Google</button>
    </div>
  </div>
</template>

<script>
import Vue from "vue";
import firebase from "firebase/app";
import "firebase/auth";
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
const provider = new firebase.auth.GoogleAuthProvider();
const store = app.firestore();

export default {
  name: "App",
  components: {
    Submission
  },
  created() {
    firebase.auth().onAuthStateChanged(user => {
      if (
        user &&
        (user.email === "bradspencer@covenantchristian.org" ||
          user.email === "andygossel@covenantchristian.org" ||
          user.email === "davidpfeifer@covenantchristian.org" ||
          user.email === "suestern@covenantchristian.org" ||
          user.email === "allysonsloan@covenantchristian.org")
      ) {
        this.user = user.uid;
        this.displayName = user.displayName;
        store
          .collection("athletic_participation")
          .orderBy("started", "desc")
          .onSnapshot(snapshot => {
            snapshot.docChanges().forEach(change => {
              if (change.type === "added") {
                this.submissions.push({
                  id: change.doc.id,
                  ...change.doc.data()
                });
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
      }
    });
  },
  computed: {
    filteredSubmissions() {
      let filtered = [];
      if (this.filter.length > 2) {
        filtered = this.submissions.filter(s =>
          s.student.toLowerCase().includes(this.filter.toLowerCase())
        );
      } else {
        filtered = this.submissions;
      }
      if (!this.showEventLink) {
        filtered = filtered.filter(f => !f.eventlink);
      }
      return filtered;
    }
  },
  data: () => {
    return {
      displayName: "",
      filter: "",
      showEventLink: false,
      submissions: [],
      user: null
    };
  },
  methods: {
    eventLink(id) {
      store.doc(`athletic_participation/${id}`).set(
        {
          eventlink: true
        },
        { merge: true }
      );
    },
    google() {
      firebase.auth().signInWithPopup(provider);
    },
    removeItem(id) {
      store.doc(`athletic_participation/${id}`).delete();
    },
    updatePage(page) {
      store.doc(`athletic_participation/${page.id}`).set(
        {
          [page.num]: page.val
        },
        { merge: true }
      );
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
