<template>
  <div>
    {{ item.year }}
    {{ item.student }}
    <img :src="page2" style="height: 50px;" />
  </div>
</template>

<script>
import AWS from 'aws-sdk'
import AWSconfig from '@/components/AWS-config'

const endpoint = new AWS.Endpoint('nyc3.digitaloceanspaces.com')
const s3 = new AWS.S3({
  endpoint,
  accessKeyId: AWSconfig.key,
  secretAccessKey: AWSconfig.secret,
})

function encode(data) {
  var str = data.reduce(function(a, b) {
    return a + String.fromCharCode(b)
  }, '')
  return btoa(str).replace(/.{76}(?=.)/g, '$&\n')
}

export default {
  data: () => {
    return {
      page2: null,
    }
  },
  props: {
    item: {
      type: Object,
      required: true,
    },
  },
  mounted() {
    if (this.item.p2) {
      console.log('get image')
      s3.getObject({ Bucket: 'covenant', Key: this.item.p2 }, (err, data) => {
        if (!err) {
          const img = encode(data.Body)
          this.page2 = `data:image/jpeg;base64,${img}`
        }
      })
    }
  },
}
</script>

<style scoped></style>
