<template>
  <div class="parent">
    <div v-if="!item.eventlink" class="child">
      <button @click="eventLink">
        EventLink
        <br />Updated
      </button>
    </div>
    <div class="child">
      {{ item.student }}
      <br />
      ({{ item.parent }})
    </div>
    <div :class="enrollment">
      Enrollment
      <br />Forms
    </div>
    <div :class="ihsaa">
      <span v-html="ihsaaType"></span>
      <div v-if="ihsaaType.length > 5">
        <button @click="loadIHSAA">View</button>
      </div>
    </div>
    <div :class="handbook">Handbook</div>
    <div :class="concussion">Concussion/SCA</div>
    <div :class="transportation">Transportation</div>
    <div v-if="!item.finished" class="child">{{ item.started.toDate() | date }}</div>
    <div v-else class="child">{{ item.finished.toDate() | date }}</div>
    <div class="child">
      <button @click="remove">Remove</button>
    </div>
  </div>
</template>

<script>
import moment from "moment";
import AWS from "aws-sdk";
import AWSconfig from "@/components/AWS-config";

const endpoint = new AWS.Endpoint("nyc3.digitaloceanspaces.com");
const s3 = new AWS.S3({
  endpoint,
  accessKeyId: AWSconfig.key,
  secretAccessKey: AWSconfig.secret
});

// eslint-disable-next-line no-unused-vars
function encode(data) {
  var str = data.reduce(function(a, b) {
    return a + String.fromCharCode(b);
  }, "");
  return btoa(str).replace(/.{76}(?=.)/g, "$&\n");
}

async function loadImage(Key) {
  try {
    const data = await s3.getObject({ Bucket: "covenant", Key }).promise();
    const encoded = encode(data.Body);
    return `data:image/jpeg;base64,${encoded}`;
  } catch (e) {
    return "";
  }
}

export default {
  computed: {
    concussion() {
      return this.item.concussion_sca_parent && this.item.concussion_sca_student
        ? "child green"
        : "child red";
    },
    enrollment() {
      return this.item.enrollment_forms ? "child green" : "child red";
    },
    handbook() {
      return this.item.handbook_parent1 &&
        this.item.handbook_parent2 &&
        this.item.handbook_student
        ? "child green"
        : "child red";
    },
    ihsaa() {
      let ihsaa = "child red";
      if (
        this.item.p1 &&
        this.item.p2 &&
        this.item.p3 &&
        this.item.p4 &&
        this.item.p5
      )
        ihsaa = "child green";
      else if (
        this.item.p1 &&
        this.item.p2 &&
        !this.item.p3 &&
        !this.item.p4 &&
        !this.item.p5
      )
        ihsaa = "child green";
      return ihsaa;
    },
    ihsaaType() {
      let type = "IHSAA";
      if (
        this.item.p1 &&
        this.item.p2 &&
        this.item.p3 &&
        this.item.p4 &&
        this.item.p5
      )
        type = "IHSAA<br>Full";
      else if (
        this.item.p1 &&
        this.item.p2 &&
        !this.item.p3 &&
        !this.item.p4 &&
        !this.item.p5
      )
        type = "IHSAA<br>Supplemental";
      else if (
        this.item.p1 ||
        this.item.p2 ||
        this.item.p3 ||
        this.item.p4 ||
        this.item.p5
      )
        type = "IHSAA<br>Partial";
      return type;
    },
    transportation() {
      return this.item.transportation ? "child green" : "child red";
    }
  },
  filters: {
    date(value) {
      return moment(value).format("MM/DD/YYYY");
    }
  },
  methods: {
    async loadIHSAA() {
      let page1 = "";
      let page2 = "";
      let page3 = "";
      let page4 = "";
      let page5 = "";
      if (this.item.p1) {
        page1 = await loadImage(this.item.p1);
      }
      if (this.item.p2) {
        page2 = await loadImage(this.item.p2);
      }
      if (this.item.p3) {
        page3 = await loadImage(this.item.p3);
      }
      if (this.item.p4) {
        page4 = await loadImage(this.item.p4);
      }
      if (this.item.p5) {
        page5 = await loadImage(this.item.p5);
      }
      let pageContent = "<html><head><title>IHSAA Form</title></head><body>";
      if (page1 !== "")
        pageContent += `<img src="${page1}" style="width: 100%;" />`;
      if (page2 !== "")
        pageContent += `<img src="${page2}" style="width: 100%;" />`;
      if (page3 !== "")
        pageContent += `<img src="${page3}" style="width: 100%;" />`;
      if (page4 !== "")
        pageContent += `<img src="${page4}" style="width: 100%;" />`;
      if (page5 !== "")
        pageContent += `<img src="${page5}" style="width: 100%;" />`;
      pageContent += "</body></html>";
      let tab = window.open("");
      tab.document.open();
      tab.document.write(pageContent);
      tab.document.close();
    },
    eventLink() {
      this.$emit("eventlink", this.item.id);
    },
    remove() {
      this.$emit("remove", this.item.id);
    }
  },
  props: {
    item: {
      type: Object,
      required: true
    }
  }
};
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
  flex-wrap: wrap;
  margin-bottom: 1vw;
}
.red {
  color: red;
}
</style>
