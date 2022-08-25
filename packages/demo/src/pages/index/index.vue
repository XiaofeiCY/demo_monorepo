<template>
    <view class="index">
        <Counter />
        <beautyUpload></beautyUpload>
        <!--        <aaaa></aaaa>-->
    </view>
</template>

<script>
import './index.less';
import Counter from '../../components/Counter.vue';
import { ref } from 'vue';
import beautyUpload from 'taro_beauty_upload';

export default {
    name: 'Index',
    components: {
        Counter,
        beautyUpload,
    },
    setup() {
        const s = ref(11);
    },
};
</script>
