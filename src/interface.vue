<script lang="ts">

import {useApi, useExtensions} from "@directus/extensions-sdk";
import {defineComponent} from "@vue/runtime-core";
import { computed, ref, watch } from 'vue';

function getRootPath(): string {
  const path = window.location.pathname
  const parts = path.split('/')
  const adminIndex = parts.indexOf('admin')
  return parts.slice(0, adminIndex).join('/') + '/';
}
function addQueryToPath(path: string, query: Record<string, string>): string {
  const queryParams = new URLSearchParams(path.split('?')[1] || '');
  for (const [key, value] of Object.entries(query)) {
    queryParams.set(key, value);
  }
  return path.split('?')[0] + '?' + queryParams;
}

function addTokenToURL(url: string, token?: string): string {
  return token ? addQueryToPath(url, { access_token: token }) : url;
}

function getToken(api: any): string {
  return (
      api.defaults?.headers?.['Authorization']?.split(' ')[1] ||
      api.defaults?.headers?.common?.['Authorization']?.split(' ')[1] ||
      null
  );
}

export default defineComponent({
  setup(props, {attrs, emit}) {
    const api = useApi();
    const fileId = ref(attrs.value)
    const fieldName = ref(attrs.field)
    const loading = ref(true);

    watch(() => {
      fileId.value = attrs.value;
      fieldName.value = attrs.field;
    });

    const src = computed(() => {
      if (!fileId.value) return null;
      const token = getToken(api);
      return addTokenToURL(window.location.origin + getRootPath() + `assets/${fileId.value}`, token);
    });
    console.log(src)

    const title = computed(() => {
      return fieldName.value
    })

    // drawer toggle
    const previewDrawerActive = ref(false);

    // check if FileComponent
    const { interfaces } = useExtensions();
    const { component: FileComponent } = interfaces.value.find(
        (i) => i.id === "file"
    );

    return {
      attrs,
      FileComponent,
      previewDrawerActive,
      src,
      title,
      loading,
    };
  },
  methods: {
    refreshIframe(){
      const iframe = this.$refs.iframe as HTMLIFrameElement;
      if (iframe) {
        this.loading = true;
        iframe.src = iframe.src; // Reload the iframe
      }
    },
    handleIframeLoad() {
      this.loading = false;
    }
  }
})

</script>

<template>
  <component :is="FileComponent" v-bind="attrs" />
  <p class="preview-ext" @click.stop="previewDrawerActive = true">Preview</p>

  <v-drawer
      v-if="src"
      v-model:modelValue="previewDrawerActive"
      subtitle="Document Preview"
      :title="title"
      icon="quick_reference"
      persistent
      @cancel="previewDrawerActive = false"
  >
    <template #actions:prepend>
      <v-button
          class="cancel"
          v-tooltip.bottom="'Refresh'"
          icon rounded primary
          @click.stop="refreshIframe()"
      >
        <v-icon name="refresh" />
      </v-button>
    </template>

    <span v-if="loading" class="loader"></span>

    <iframe
        class="doc"
        :src="`https://docs.google.com/gview?embedded=true&url=${src}`"
        @load="handleIframeLoad"
        ref="iframe"
    >
    </iframe>


  </v-drawer>
</template>

<style scoped>
/* Add your styles here if needed */
.preview-ext {
  color: var(--project-color);
  cursor: pointer;
}

.doc {
  width: 100%;
  height: 100vh;
}

.loader {
  width: 16px;
  height: 16px;
  border-radius: 50%;
  display: block;
  margin:15px auto;
  position: relative;
  background: #FFF;
  box-shadow: -24px 0 #FFF, 24px 0 #FFF;
  box-sizing: border-box;
  animation: shadowPulse 2s linear infinite;
}

@keyframes shadowPulse {
  33% {
    background: #FFF;
    box-shadow: -24px 0 var(--project-color), 24px 0 #FFF;
  }
  66% {
    background: var(--project-color);
    box-shadow: -24px 0 #FFF, 24px 0 #FFF;
  }
  100% {
    background: #FFF;
    box-shadow: -24px 0 #FFF, 24px 0 var(--project-color);
  }
}
</style>