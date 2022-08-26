<template>
    <view class="index">
        123
        <BeautyUpload :preImgList="[]" :url="uploadSrc"></BeautyUpload>
    </view>
</template>

<script setup>
// import './index.less';
// import Counter from '../../components/Counter.vue';
import { ref } from 'vue';
import BeautyUpload from 'taro_beauty_upload';

const uploadSrc = ref('http://120.79.155.40/uc/file/manage/upload');
</script>
