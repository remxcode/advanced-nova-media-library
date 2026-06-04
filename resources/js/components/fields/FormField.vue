<template>
  <component v-if="currentlyIsVisible" :is="currentField.fullSize ? 'FullWidthField' : 'DefaultField'" :field="currentField" :errors="errors" :show-help-text="showHelpText">
    <template #field>
      <div :class="{'px-8 pt-6': currentField.fullSize}">
        <gallery slot="value" ref="gallery" v-if="hasSetInitialValue"
                 :modelValue="value" @update:modelValue="handleChange" :editable="!currentlyIsReadonly" :removable="currentField.removable" custom-properties :field="currentField" :multiple="currentField.multiple" :uploads-to-vapor="currentField.uploadsToVapor"
                 :has-error="hasError" :first-error="firstError"/>

        <div v-if="currentField.existingMedia && !currentlyIsReadonly">
          <Button
            class="mt-2"
            icon="arrows-pointing-out"
            :label="openExistingMediaLabel"
            @click.prevent="existingMediaOpen = true"
            />
          <existing-media :open="existingMediaOpen" @close="existingMediaOpen = false" @select="addExistingItem"/>
        </div>
        <help-text
          class="error-text mt-2 text-danger"
          v-if="showErrors && hasError"
        >
          {{ firstError }}
        </help-text>
      </div>
    </template>
  </component>
</template>

<script>
import {DependentFormField, HandlesValidationErrors} from 'laravel-nova';
import Gallery from '../Gallery';
import FullWidthField from '../FullWidthField';
import ExistingMedia from '../ExistingMedia';
import objectToFormData from 'object-to-formdata';
import get from 'lodash/get';
import {Button} from "laravel-nova-ui";

export default {
  mixins: [DependentFormField, HandlesValidationErrors],
  components: {
    Button,
    Gallery,
    FullWidthField,
    ExistingMedia
  },
  props: ['resourceName', 'resourceId', 'field'],
  data() {
    return {
      hasSetInitialValue: false,
      existingMediaOpen: false,
    }
  },
  computed: {
    openExistingMediaLabel() {
      const type = this.currentField.type === 'media' ? 'Media' : 'File';

      if (this.currentField.multiple || this.value.length === 0) {
        return this.__(`Add Existing ${type}`);
      }

      return this.__(`Use Existing ${type}`);
    },

    normalizedMediaValue() {
      return (this.value || []).map((item) => {
        if (!item || typeof item !== 'object') {
          return null;
        }

        return ['id', 'uuid', 'name', 'file_name'].reduce((media, key) => {
          if (item[key] !== undefined && item[key] !== null && item[key] !== '') {
            media[key] = item[key];
          }

          return media;
        }, {});
      }).filter((item) => item && Object.keys(item).length > 0);
    },

    currentFieldValues() {
      return {
        [this.fieldAttribute]: this.normalizedMediaValue,
      };
    },
  },
  methods: {
    /*
     * Set the initial, internal value for the field.
     */
    setInitialValue() {
      let value = this.currentField.value || [];

      if (!this.currentField.multiple) {
        value = value.slice(0, 1);
      }

      this.value = value;
      this.hasSetInitialValue = true;
    },

    /**
     * Fill the given FormData object with the field's internal value.
     */
    fill(formData) {
      if (!this.currentlyIsVisible) {
        return;
      }

      const field = this.fieldAttribute;
      this.value.forEach((file, index) => {
        const isNewImage = !file.id;

        if (isNewImage) {
          if (file.isVaporUpload) {
            // In case of Vapor upload, do not send the file's binary data over the wire.
            // The file can already be found in the bucket.
            formData.append(`__media__[${field}][${index}][is_vapor_upload]`, true);
            formData.append(`__media__[${field}][${index}][key]`, file.vaporFile.key);
            formData.append(`__media__[${field}][${index}][uuid]`, file.vaporFile.uuid);
            formData.append(`__media__[${field}][${index}][file_name]`, file.vaporFile.filename);
            formData.append(`__media__[${field}][${index}][file_size]`, file.vaporFile.file_size);
            formData.append(`__media__[${field}][${index}][mime_type]`, file.vaporFile.mime_type);
          } else {
            formData.append(`__media__[${field}][${index}]`, file.file, file.name);
          }
        } else {
          formData.append(`__media__[${field}][${index}]`, file.id);
        }

        objectToFormData({
          [`__media-custom-properties__[${field}][${index}]`]: this.getImageCustomProperties(file)
        }, {}, formData);
      });
    },

    getImageCustomProperties(image) {
      return (this.currentField.customPropertiesFields || []).reduce((properties, {attribute: property}) => {
        properties[property] = get(image, `custom_properties.${property}`);

        // Fixes checkbox problem
        if (properties[property] === true) {
          properties[property] = 1;
        }

        return properties;
      }, {})
    },

    /**
     * Update the field's internal value.
     */
    handleChange(value) {
      this.setValueAndEmit(value);
    },

    setValueAndEmit(value) {
      this.value = value;
      this.emitFieldValueChange(this.fieldAttribute, this.normalizedMediaValue);
      this.$emit('field-changed');
    },

    addExistingItem(item) {
      // Copy to trigger watcher to recognize differnece between new and old values
      // https://github.com/vuejs/vue/issues/2164
      let copiedArray = this.value.slice(0)

      if (!this.currentField.multiple) {
        copiedArray.splice(0, 1);
      }

      copiedArray.push(item);
      this.setValueAndEmit(copiedArray);
    }
  },
};
</script>
