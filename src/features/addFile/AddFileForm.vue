<script lang="ts" setup>
import { ref } from 'vue';
import type { DirectoryEntryRef, FileEntryRef } from '../../entities/entry';

const props = defineProps<{
  directoryEntry: DirectoryEntryRef;
}>();

const emit = defineEmits<{
  written: [fileEntry: FileEntryRef];
  cancel: [];
}>();

const fileName = ref<string>();

const stateFile = ref<File>();

const loading = ref(0);

const onSubmit = async () => {
  loading.value += 1;
  try {
    if (fileName.value?.length) {
      const fileEntry = await props.directoryEntry.writeFile(
        fileName.value,
        stateFile.value,
      );
      emit('written', fileEntry);
    }
  } finally {
    loading.value -= 1;
  }
};

const onClickCancel = () => {
  fileName.value = undefined;
  stateFile.value = undefined;

  emit('cancel');
};

const onChangeFileInput = ({ target }: Event) => {
  if (target instanceof HTMLInputElement) {
    const { files } = target;

    if (files?.length) {
      stateFile.value = files.item(0) ?? undefined;
    }
  }
};
</script>

<template>
  <form
    class="block-spacing is-flex is-flex-direction-column"
    @submit.prevent="onSubmit"
  >
    <div class="field">
      <label class="label">File name</label>

      <div class="control">
        <input
          v-model="fileName"
          class="input"
          type="text"
          placeholder="file name"
        />
      </div>
    </div>

    <div
      class="file"
      :class="{
        'has-name': stateFile?.name,
      }"
    >
      <label class="file-label">
        <input
          class="file-input"
          type="file"
          name="file"
          @change="onChangeFileInput"
        />

        <span class="file-cta">
          <span class="file-icon">
            <i class="fas fa-upload" />
          </span>

          <span class="file-label"> Choose a file… </span>
        </span>

        <span v-if="stateFile?.name" class="file-name">
          {{ stateFile.name }}
        </span>
      </label>
    </div>

    <div class="field is-grouped">
      <div class="control">
        <button class="button" type="submit" :class="{ 'is-loading': loading }">
          Add
        </button>
      </div>

      <div class="control">
        <button
          class="button"
          type="button"
          :disabled="!!loading"
          @click="onClickCancel"
        >
          Cancel
        </button>
      </div>
    </div>
  </form>
</template>
