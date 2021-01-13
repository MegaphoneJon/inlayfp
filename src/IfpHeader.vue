<template>
  <div :is="modifier" :class="content">{{headerText}}</div>
</template>

<script>

export default {
  // 'content' (String) form processor input name.
  props: ['content'],
  data() {
    const d = {
      myId: this.$root.getNextId()
    };
    return d;
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

