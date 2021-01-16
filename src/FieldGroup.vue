<template>
  <div :class="groupClass">
    <div v-for="(item, idx) in fields"
         :is="item.tag"
         :key="idx"
         :content="item.content"
         :groupClass="getGroupClass(item.tag)"
         >
    </div>
  </div>
</template>
<script>

import IfpField from './IfpField.vue';
import IfpHeader from './IfpHeader.vue';

export default {
  name: 'FieldGroup',
  props: ['content', 'groupClass'],
  components: {IfpField, IfpHeader},
  data() {
    // content is an array of items that describe other components.
    // This allows nested FieldGroup items as well as various types of form
    // input, currently all handled by the IfpField component (as there's a lot of
    // crossover between types).
    return {
      fields: this.content,
    }
  },
  methods: {
    getGroupClass: function (tag) {
      switch (tag){
        case 'FieldGroup':
          return 'ifp-group'
        case 'IfpHeader':
          return 'ifp-header';
        default:
          return 'ifp-field';
      }
    }
  }
}

</script>
<style lang="scss">
// Responsive 2 column layout
.ifp-group {
  margin-left: -1rem;
  margin-right: -1rem;
  display: flex;
  flex-wrap: wrap;
  &>* {
    padding:0 1rem;
    flex: 1 0 14rem;
  }
}
</style>

