<template>
    <div class="container">
      <h2>Upload file</h2>
      <vue-base64-file-upload 
        class="v1"
        accept="image/png,image/jpeg"
        image-class="v1-image"
        input-class="v1-input"
        @size-exceeded="onSizeExceeded"
        @file="onFile"
        @load="onLoad" />
    </div>
</template>

<script>
import VueBase64FileUpload from 'vue-base64-file-upload';

export default {
  props: ['content'],
  components: {VueBase64FileUpload},
  data() {
    const d = {
      myId: this.$root.getNextId()
    };
    return d;
  },
  methods: {
    onFile(file) {
      // Form Processor expects an array with values "name", "content", and "mime_type".
      // We get the name now, and the rest onLoad in a moment.
      this.$root.values[this.def.name] = {};
      this.$root.values[this.def.name].name = file.name;
    },

    onLoad(dataUri) {
      // Parse the data URL.
      var matches = dataUri.match(/^data:(.+);base64,(.*)$/);
      this.$root.values[this.def.name].content = matches[2];
      this.$root.values[this.def.name].mime_type = matches[1];
    },

    onSizeExceeded(size) {
      alert(`Image ${size}Mb size exceeds the 10 megabyte limit.`);
    }
  },
  computed: {
    def() {
      // The definition for the header we're handling.
      return this.$root.inlay.initData.fieldDefs[this.content];
    },
    modifier() {
      return this.def.modifier;
    },
    headerText() {
      return this.def.name.replace('_', ' ');
    }
  },
  created() {
    this.$root.inlay.initData.fieldDefs[this.content].include = true;
  }
}

</script>

