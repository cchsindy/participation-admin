<template>
  <div>
    <div v-if="orphans.length > 0">
      {{ currentIndex + 1 }} of {{ orphans.length }} orphans found.
      <br />
      {{ orphans[currentIndex].id }}
      [{{ orphans[currentIndex].keys.length }} page(s)]
      <button
        @click="viewOrphan"
      >View</button>
      <br />
      <button @click="previous">Previous</button>&nbsp;&nbsp;
      <button @click="next">Next</button>
      <br />
      <br />
    </div>
    <div v-if="orphans.length === 0">
      <button @click="loadOrphans">Show Orphans</button>
      <br />
      <br />
    </div>
  </div>
</template>

<script>
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

// eslint-disable-next-line no-unused-vars
async function loadImage(Key) {
  try {
    const data = await s3.getObject({ Bucket: "covenant", Key }).promise();
    const encoded = encode(data.Body);
    return `data:image/jpeg;base64,${encoded}`;
  } catch (e) {
    return "";
  }
}

// eslint-disable-next-line no-unused-vars
async function createHTML(orphans) {
  try {
    let html = "";
    for (const orphan of orphans) {
      const img = await loadImage(orphan);
      html += `<img src="${img}" style="width: 100%;" />`;
    }
    return html;
  } catch (e) {
    return "";
  }
}

export default {
  data: () => {
    return {
      currentIndex: 0,
      orphans: []
    };
  },
  methods: {
    loadOrphans() {
      s3.listObjects({ Bucket: "covenant" }, (err, data) => {
        if (!err) {
          const orphans = [];
          let prevKey = "";
          for (const c of data.Contents) {
            if (c.Key.substring(0, 5) === "ihsaa") {
              let k = c.Key.substring(6, c.Key.length - 10);
              if (k.length > 6 && !this.subids.includes(k)) {
                if (k === prevKey) {
                  orphans[orphans.length - 1].keys.push(c.Key);
                } else {
                  orphans.push({
                    id: k,
                    keys: [c.Key]
                  });
                }
                prevKey = k;
              }
            }
          }
          this.orphans = orphans;
        }
      });
    },
    next() {
      this.currentIndex++;
      if (this.currentIndex === this.orphans.length) this.currentIndex--;
    },
    previous() {
      this.currentIndex--;
      if (this.currentIndex < 0) this.currentIndex = 0;
    },
    viewOrphan() {
      createHTML(this.orphans[this.currentIndex].keys).then(raw => {
        let pageContent = "<html><head><title>IHSAA Form</title></head><body>";
        pageContent += raw;
        pageContent += "</body></html>";
        let tab = window.open("");
        tab.document.open();
        tab.document.write(pageContent);
        tab.document.close();
      });
    }
  },
  props: {
    subids: {
      type: Array,
      required: true
    }
  }
};
</script>

<style scoped>
</style>