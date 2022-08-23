<template>
  <div class="file-graphical">
    <div
      v-if="values.children?.length !== 0"
      class="file-warpper mt-4 flex flex-wrap justify-start"
    >
      <div
        v-for="file in values.children"
        :key="file.id"
        class="flex flex-col justify-end items-center w-18 px-1 mx-2 mb-5 transition-all duration-100 hover:bg-gray-100 hover:shadow hover:rounded-lg animate__animated animate__fadeIn faster"
      >
        <Folder
          v-if="file?.type === '文件夹' && file.path === '' && file.size === 0"
          class="w-10 text-primary cursor-pointer"
          @click="handleExpandedKeys(file)"
        />
        <div
          class="image-item-wrapper w-18 relative"
          v-else-if="file?.type === '图片'"
          @click="handleSelectedKeys(file)"
        >
          <n-image
            class="shadow-lg rounded"
            lazy
            :src="file.path"
            width="70"
            object-fit="cover"
            preview-disabled
            fallback-src="./src/assets/logo.png"
            :intersection-observer-options="{
              root: '#image-scroll-container',
            }"
            :on-load="handleOnLoad"
          />
          <div
            v-if="isLoadingImg"
            class="z-10 w-18 h-12 absolute right-0 top-0 left-0"
          >
            <n-skeleton width="100%" height="100%" :sharp="false" />
          </div>
        </div>
        <video
          v-else-if="file.type === '视频文件'"
          class="shadow-md rounded"
          width="70"
          style="max-height: 46px"
          @click="handleSelectedKeys(file)"
        >
          <source :src="file.path" type="video/mp4" />
        </video>
        <FileTraySharp
          class="w-10 text-primary cursor-pointer"
          v-else-if="file.type === '压缩文件'"
          @click="handleSelectedKeys(file)"
        />
        <DocumentTextOutline
          v-else
          class="w-10 text-gray-400 cursor-pointer"
          @click="handleSelectedKeys(file)"
        />
        <n-tooltip
          :show-arrow="false"
          placement="bottom"
          trigger="hover"
          :style="{ backgroundColor: 'white', color: 'black' }"
        >
          <template #trigger>
            <p
              class="self-center truncate w-18 mt-1 text-center text-xs"
              @click="handleSelectedKeys(file)"
            >
              <ShowOrEdit
                truncate
                :value="file.name"
                :onUpdateValue="handleUpadeteName"
              />
            </p>
          </template>
          {{ file.name }}
        </n-tooltip>
      </div>
    </div>
    <!-- <div class="w-18 h-10">
      <n-skeleton width="100%" height="100%" :sharp="false" />
    </div> -->
    <Empty v-else description="空文件夹"> </Empty>
    <DropDown
      :show="showDropdownRef"
      :position="{ x: xRef, y: yRef }"
      @select="handleSelectDropDownItem"
      @clickoutside="handleClidkOutside"
    />
  </div>
</template>

<script setup lang="ts">
import { PropType, ref, toRefs, defineAsyncComponent } from "vue";
import { NImage, NSkeleton, NTooltip } from "naive-ui";
import { useFiles } from "@/hooks/useFiles";
import { FileListData } from "@/models/file";
import { Folder, DocumentTextOutline, FileTraySharp } from "@vicons/ionicons5";
const DropDown = defineAsyncComponent(
  () => import("@/components/commons/drop-down/index.vue")
);
const ShowOrEdit = defineAsyncComponent(() => import("./file-edit.vue"));
const Empty = defineAsyncComponent(
  () => import("@/components/commons/empty/index.vue")
);

const props = defineProps({
  values: {
    type: Object as PropType<FileListData>,
    required: true,
    default: {},
  },
});
const emits = defineEmits(["selectedKeys", "expandedKeys"]);
const { onUpdateFileName } = useFiles();
const showDropdownRef = ref(false);
const xRef = ref(0);
const yRef = ref(0);
const currentFileRef = ref<FileListData | null>(null);
const isLoadingImg = ref(true);

const handleSelectedKeys = (file: FileListData) => {
  currentFileRef.value = file;
  emits("selectedKeys", file);
};
const handleExpandedKeys = (file: FileListData) => {
  emits("expandedKeys", file);
};
const handleUpadeteName = (v: string) => {
  if (
    v &&
    v !== currentFileRef.value?.name &&
    currentFileRef.value &&
    currentFileRef.value.identity
  ) {
    currentFileRef.value.name = v;
    onUpdateFileName({
      identity: currentFileRef.value.identity,
      name: v,
    });
  }
};
const handleSelectDropDownItem = (value: string) => {
  console.log("--drop select", value);
  showDropdownRef.value = false;
};
const handleClidkOutside = (value: boolean) => {
  showDropdownRef.value = value;
};
const handleOnLoad = (e: Event) => {
  isLoadingImg.value = false;
};

toRefs(props);
</script>

<style lang="scss" scoped>
.file-graphical {
  .image-item-wrapper {
    max-height: 46px;
  }
}
</style>