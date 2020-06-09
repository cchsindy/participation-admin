<template>
  <div class="parent">
    <div class="child">
      {{ item.student }}
      <br />
      ({{ item.parent }})
    </div>
    <div :class="enrollment">Enrollment</div>
    <div :class="ihsaa" v-html="ihsaaType"></div>
    <div :class="handbook">Handbook</div>
    <div :class="concussion">Concussion/SCA</div>
    <div :class="transportation">Transporation</div>
    <div v-if="!item.finished" class="child">
      {{ item.started.toDate() | date }}
    </div>
    <div v-else class="child">{{ item.finished.toDate() | date }}</div>
  </div>
</template>

<script>
import moment from 'moment'
// import AWS from "aws-sdk";
// import AWSconfig from "@/components/AWS-config";

// const endpoint = new AWS.Endpoint("nyc3.digitaloceanspaces.com");
// const s3 = new AWS.S3({
//   endpoint,
//   accessKeyId: AWSconfig.key,
//   secretAccessKey: AWSconfig.secret
// });

// function encode(data) {
//   var str = data.reduce(function(a, b) {
//     return a + String.fromCharCode(b);
//   }, "");
//   return btoa(str).replace(/.{76}(?=.)/g, "$&\n");
// }

export default {
  computed: {
    concussion() {
      return this.item.concussion_sca_parent && this.item.concussion_sca_student
        ? 'child green'
        : 'child red'
    },
    enrollment() {
      return this.item.enrollment_forms ? 'child green' : 'child red'
    },
    handbook() {
      return this.item.handbook_parent1 &&
        this.item.handbook_parent2 &&
        this.item.handbook_student
        ? 'child green'
        : 'child red'
    },
    ihsaa() {
      let ihsaa = 'child red'
      if (
        this.item.p1 &&
        this.item.p2 &&
        this.item.p3 &&
        this.item.p4 &&
        this.item.p5
      )
        ihsaa = 'child green'
      else if (
        this.item.p1 &&
        this.item.p2 &&
        !this.item.p3 &&
        !this.item.p4 &&
        !this.item.p5
      )
        ihsaa = 'child green'
      return ihsaa
    },
    ihsaaType() {
      let type = 'IHSAA'
      if (
        this.item.p1 &&
        this.item.p2 &&
        this.item.p3 &&
        this.item.p4 &&
        this.item.p5
      )
        type = 'IHSAA<br>Full'
      else if (
        this.item.p1 &&
        this.item.p2 &&
        !this.item.p3 &&
        !this.item.p4 &&
        !this.item.p5
      )
        type = 'IHSAA<br>Supplemental'
      else if (
        this.item.p1 ||
        this.item.p2 ||
        this.item.p3 ||
        this.item.p4 ||
        this.item.p5
      )
        type = 'IHSAA<br>Partial'
      return type
    },
    transportation() {
      return this.item.transportation ? 'child green' : 'child red'
    },
  },
  filters: {
    date(value) {
      return moment(value).format('MM/DD/YYYY')
    },
  },
  props: {
    item: {
      type: Object,
      required: true,
    },
  },
  // mounted() {
  //   if (this.item.p2) {
  //     console.log("get image");
  //     s3.getObject({ Bucket: "covenant", Key: this.item.p2 }, (err, data) => {
  //       if (!err) {
  //         const img = encode(data.Body);
  //         this.page2 = `data:image/jpeg;base64,${img}`;
  //       }
  //     });
  //   }
  // }
}
</script>

<style scoped>
.child {
  border: 2px dashed #ccc;
  padding: 0.5vw;
}
.green {
  color: green;
}
.parent {
  display: flex;
  margin-bottom: 1vw;
}
.red {
  color: red;
}
</style>
