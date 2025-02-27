<template>
  <n-card title="Tags" :bordered="false" class="minimal left" v-if="isLoaded">
    <template #header-extra v-if="store.hasRights()">
      <n-button v-if="!showForm" type="primary" @click="showForm = true">+ new</n-button>
    </template>

    <n-input-group style="display: block; max-width: 300px; text-align: center; margin: 0 auto" v-if="showForm">
      <n-space vertical>
        <n-input v-model:value="newTag.title" type="text" placeholder="Title" />
        <!-- <n-input v-model:value="newTag.ru" type="text" placeholder="Russian title" /> -->

        <n-space justify="center">
          <n-button tertiary type="warning" @click="showForm = false">Cancel</n-button>
          <n-button type="info" @click="editTag(newTag)">Save</n-button>
        </n-space>
      </n-space>
      <n-divider style="width: 300px; text-align: center; margin: auto; padding: 1rem" />
    </n-input-group>

    <div v-if="!tags.length">
      <n-text type="error">There are no tags!</n-text>
    </div>

    <n-space vertical size="large">
      <n-grid x-gap="12" cols="2" y-gap="6" responsive="screen" v-for="item in tags" :key="item.id">
        <n-gi>
          <n-input v-model:value="item.title" placeholder="Title" v-if="store.hasRights()" />
          <n-tag type="primary" size="large" v-else>{{ item.title }}</n-tag>
        </n-gi>
        <n-gi style="text-align: right" v-if="store.hasRights()">
          <n-dropdown trigger="hover" :options="[
    { label: 'Save', key: 0, data: item },
    { label: 'Delete', key: 1, data: item },
  ]" @select="handleSelect">
            <n-button>Manage</n-button>
          </n-dropdown>
        </n-gi>
        <!-- <n-gi>
          <n-input v-model:value="item.ru" placeholder="English title" />
        </n-gi> -->
        <!-- <n-gi style="text-align: right">
          <n-button type="info" @click="editTag(item)">Save</n-button>
        </n-gi> -->
      </n-grid>
    </n-space>
  </n-card>
</template>

<script setup lang="ts">
import store from '../store';
import { ref, reactive, onBeforeMount } from 'vue';
import { useMessage } from 'naive-ui';

const message = useMessage();
const newTag = reactive({ title: '' });
const tags: Array<ITag> = reactive([] as Array<ITag>);
const showForm = ref(false);
const isLoaded = ref(false);

onBeforeMount(async () => {
  const data = await store.get('tags');
  Object.assign(tags, data);
  // console.log('data from server', data);
  isLoaded.value = true;
});

const handleSelect = async (key: string | number, option: any) => {
  // console.log(String(key), option);
  if (key) {
    const { data } = await store.deleteById('tags', option.data.id);
    if (data.success) {
      message.success('The tag was deleted succesfully');

      Object.assign(
        tags,
        tags.filter((x: ITag) => x.id !== data?.id)
      );
    } else {
      message.error(`The tag cannot be deleted.\nThere are comments bound (${data?.comments})`);
    }
  } else {
    await editTag(option.data);
  }
};

const editTag = async (tag: ITag) => {
  // console.log('edit tag', tag);
  const params: ITag = {} as ITag;
  if (tag?.id) {
    params.id = tag.id;
  }
  params.title = tag.title;
  const result = await store.post('tag', params);
  if (result?.data?.id) {
    params.id = result.data.id;
    if (!tag?.id) {
      tags.unshift(params);
      newTag.title = '';
    }
  } else {
    console.log('error', result);
  }
};
</script>
