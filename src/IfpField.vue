<template>
  <div :class="content">
    <label >{{label}}<template v-if="def.is_required == 1"><span class="required-marker" title="This field is required"> *</span></template></label>
    <input
      v-if="isInputType"
      :name="def.name"
      :type="inputType"
      :ref="def.name"
      :required="def.is_required == 1"
      :disabled="submissionRunning"
      v-model="$root.values[def.name]"
      />
    <textarea
      v-if="inputType === 'textarea'"
      rows=4
      cols=40
      v-model="$root.values[def.name]"
      :required="def.is_required == 1"
      :name="def.name"
      :disabled="submissionRunning"
      :ref="def.name"
      />
    <select
      v-if="inputType === 'select'"
      v-model="$root.values[def.name]">
        <option v-for="option in def.option_values" :value="option.value">{{option.label}}</option>
    </select>

    <div v-if="inputType === 'radio'" class="ifp-radios">
      <div v-for="option in def.option_values" class="ifp-radio">
        <label :for="myId + option.value"><input
          type="radio"
          :name="def.name"
          :required="def.is_required == 1"
          :disabled="submissionRunning"
          :value="option.value"
          v-model="$root.values[def.name]"
          :id="myId + option.value" />
        {{option.label}}</label>
      </div>
    </div>

    <div v-if="inputType === 'checkbox'" class="ifp-checkboxes">
      <div v-for="(option, i) in checkboxes" class="ifp-checkbox">
        <label :for="myId + i"><input
          type="checkbox"
          :id="myId + i"
          :name="def.name"
          :required="def.is_required == 1"
          :disabled="submissionRunning"
          :value="option.value"
          v-model="option.selected"
          @change="updateValue()"
           />
        {{option.label}}</label>
      </div>
    </div>

    <div v-if="inputType === 'file'" class="ifp-file">
        <vue-base64-file-upload
        accept="image/png,image/jpeg,image/gif,image/webp,application/pdf,text/plain,text/csv,audio/mpeg"
        image-class="ifp-file-image"
        input-class="ifp-file-input"
        @size-exceeded="onFileSizeExceeded"
        @file="onFileSelected"
        @load="onFileUploaded" />
    </div>
  </div>
</template>
<style lang="scss">
// Create nice hanging indent with no dead, unclickable whitespace.
.ifp-radio>label {
  position: relative;
  $spaceForRadio: 1.6rem;
  padding-left: $spaceForRadio;
  &>input {
    position: absolute;
    margin-left: -$spaceForRadio;
  }
}
</style>
<script>

import IfpField from './IfpField.vue';
import VueBase64FileUpload from 'vue-base64-file-upload';

export default {
  // 'content' (String) form processor input name.
  props: ['content'],
  components: {VueBase64FileUpload},
  data() {
    const d = {
      myId: this.$root.getNextId(),
      checkboxes: []
    };

    // If we're handling checkboxes then we have a composite situation going on.
    if (this.$root.inlay.initData.fieldDefs[this.content].type.name === 'Checkbox') {
      this.$root.inlay.initData.fieldDefs[this.content].option_values.forEach(v => {
        d.checkboxes.push(Object.assign({'selected': false}, v));
      });
    }

    return d;
  },
  methods: {
    updateValue() {
      // Copy just the values that are selected.
      this.$root.values[this.def.name] = this.checkboxes
        .filter(option => option.selected)
        .map(option => option.value);
    },
    onFileSelected(file) {
      // Form Processor expects an array with values "name", "content", and "mime_type".
      // We get the name now, and the rest onLoad in a moment.
      this.$root.values[this.def.name] = {};
      this.$root.values[this.def.name].name = file.name;
    },

    onFileUploaded(dataUri) {
      // Parse the data URL.
      var matches = dataUri.match(/^data:(.+);base64,(.*)$/);
      this.$root.values[this.def.name].content = matches[2];
      this.$root.values[this.def.name].mime_type = matches[1];
    },

    onFileSizeExceeded(size) {
      alert(`Image ${size}Mb size exceeds the 10 megabyte limit.`);
    }
  },
  computed: {
    def() {
      // The field definition for the input we're handling.
      return this.$root.inlay.initData.fieldDefs[this.content];
    },
    submissionRunning() {
      return this.$root.submissionRunning;
    },
    inputType() {
      var typeName = this.def.type.name;

      // Email is a special case.
      if (typeName === 'String' && this.def.validators.find(v => v.validator.name === 'email')) {
        return 'email';
      }

      // Map the rest to lower case, except the following oddballs.
      return {
        'String'           : 'text',
        'Text'             : 'textarea',
        'OptionGroup'      : 'select',
        'Integer'          : 'number',
        'Float'            : 'number',
        'FormattedDecimal' : 'text',
        'Boolean'          : 'radio',
        'file_url'         : 'url',
      }[typeName] || typeName.toLowerCase();

    },
    isInputType() {
      return ['text','email','date','time','number', 'url'].includes(this.inputType);
    },
    label() {
      return this.def.title;
    }
  },
  created() {
    this.$root.inlay.initData.fieldDefs[this.content].include = true;
  }
}

</script>

