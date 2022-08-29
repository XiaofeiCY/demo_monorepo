<template>
    <view class="beauty_upload">
        123中转站
        <template v-for="(item, index) in tempFilePaths" :key="item.url">
            <view class="pic_section">
                <view @click="showPicItem(item, index)">
                    <image :class="['pic_item', 'common_item']" :src="item.url" />
                </view>
                <view v-show="item.status !== 'success'" class="img_mask">
                    <template v-if="item.status === 'uploading'">
                        <image v-if="close" :class="['pic_item', 'common_item']" :src="close" />
                        <!--                        <nut-icon name="loading1" size="24"></nut-icon>-->
                    </template>
                    <template v-else>{{ item.msg }}</template>
                </view>
                <view v-show="!disabled" class="pic_del">
                    ×
                    <!--                    <nut-icon name="close-little" @click="itemDel(item, index)"></nut-icon>-->
                </view>
            </view>
        </template>
        <view v-show="!disabled && tempFilePaths.length < 6" class="common_item" @click="uploadFile">
            <image :lazy-load="true" src="/beautyUpload/assets/camera.png" />
        </view>
        <canvas
            :style="`width:${cWidth}px;height:${cHeight}px;position: absolute;left:-1000px;top:-1000px;`"
            canvas-id="canvas"
        ></canvas>
    </view>
</template>

<script lang="ts" setup>
import './index.less';
import { ref, watch } from 'vue';
import type { UploadTask } from '@tarojs/taro';
import Taro from '@tarojs/taro';
import close from '../assets/close.png';
// import camera from '../assets/camera.png';
// import c from '.'

type chooseImageOptions = Parameters<typeof Taro.chooseImage>[0];

interface imgObj {
    url: string;
    name?: string;
    size?: number;
    status: string; // ready,uploading,success,error
    msg?: string;
}

const props = withDefaults(
    defineProps<{
        url: string;
        count?: number;
        preImgList: imgObj[];
        disabled?: boolean;
        sizeType?: chooseImageOptions['sizeType'];
        closeAll?: boolean;
    }>(),
    {
        url: '',
        count: 6,
        preImgList: () => {
            return [];
        },
        disabled: false,
        sizeType: () => {
            return ['compressed'];
        },
        closeAll: false,
    },
);

const emit = defineEmits(['uploadSuccess', 'showPicItem', 'delItem']);
const imageList = ref<imgObj[]>([]);
const tempFilePaths = ref<imgObj[]>([]);
const uploadIndex = ref(0);
const cWidth = ref(0);
const cHeight = ref(0);
const uploadTaskList = ref<
    {
        imgItem: imgObj;
        uploadItem: UploadTask | null;
    }[]
>([]);

const uploadFile = () => {
    Taro.chooseImage({
        count: props.count - tempFilePaths.value.length,
        sizeType: props.sizeType,
        success: async (res) => {
            tempFilePaths.value.push(
                ...res.tempFiles.map((item) => {
                    return {
                        url: item.path,
                        size: item.size,
                        status: 'uploading',
                        name: 'file',
                    };
                }),
            );
            console.log('res', res, uploadIndex.value, tempFilePaths.value);
            await uploadFn(tempFilePaths.value[uploadIndex.value]);
        },
    });
};
const compressionImg = (item) => {
    return new Promise((resolve, reject) => {
        if (item) {
            // -----返回选定照片的本地文件路径列表，获取照片信息-----------
            Taro.getImageInfo({
                src: item.url,
                success: async (res) => {
                    // ---------利用canvas压缩图片--------------
                    let ratio = 2;
                    let canvasWidth = res.width; // 图片原始长宽
                    let canvasHeight = res.height;
                    while (canvasWidth > 1000 || canvasHeight > 1000) {
                        // 保证宽高在1000以内
                        canvasWidth = Math.trunc(res.width / ratio);
                        canvasHeight = Math.trunc(res.height / ratio);
                        ratio++;
                    }
                    cWidth.value = canvasWidth;
                    cHeight.value = canvasHeight;

                    // ----------绘制图形并取出图片路径--------------
                    const ctx = Taro.createCanvasContext('canvas');
                    ctx.drawImage(res.path, 0, 0, canvasWidth, canvasHeight);
                    ctx.draw(false, () => {
                        const t = setTimeout(() => {
                            clearTimeout(t);
                            Taro.canvasToTempFilePath({
                                canvasId: 'canvas',
                                destWidth: canvasWidth,
                                destHeight: canvasHeight,
                                success: (res) => {
                                    resolve(res.tempFilePath);
                                },
                                fail: (res) => {
                                    reject(new Error(res.errMsg));
                                },
                            });
                        }, 100);
                    });
                },
                // 留一定的时间绘制canvas
                fail: (res) => {
                    reject(new Error(res.errMsg));
                },
            });
        } else {
            reject(new Error('图片获取失败'));
        }
    });
};
const uploadFn = async (item) => {
    if (item) {
        compressionImg(item)
            .then((res) => {
                item.url = res;
                const uploadTask = Taro.uploadFile({
                    url: props.url,
                    filePath: item.url,
                    name: 'file',
                    timeout: 6000,
                    success: (r) => {
                        if (r.statusCode === 200) {
                            item.status = 'success';
                            item.msg = '上传成功';
                            const responseText = JSON.parse(r.data);
                            imageList.value.push({
                                url: responseText.data[0],
                                name: 'file',
                                size: item.size,
                                status: item.status,
                            });
                            emit('uploadSuccess', { successData: r });
                        }
                        uploadStatusCatch(item, true);
                    },
                    fail: () => {
                        uploadTask.abort();
                        if (!props.closeAll) {
                            uploadStatusCatch(item, false);
                        }
                    },
                });
                uploadTaskList.value.push({ imgItem: item, uploadItem: uploadTask });
                // console.log('item.url', item.url);
            })
            .catch(async () => {
                await uploadStatusCatch(item, false);
            });
    }
};
const uploadStatusCatch = async (item, isUpload) => {
    if (!isUpload) {
        item.status = 'error';
        item.msg = '图片过大,上传失败';
    }
    uploadIndex.value++;
    if (uploadIndex.value !== tempFilePaths.value.length) {
        await uploadFn(tempFilePaths.value[uploadIndex.value]);
    }
};
const showPicItem = (item, index) => {
    emit('showPicItem', { fileItem: imageList.value[index], index, imageList: imageList.value });
};
const itemDel = (item, index) => {
    tempFilePaths.value.splice(index, 1);
    // 上传过程中删掉的同时需要中断被删的附件
    uploadTaskList.value[index].uploadItem?.abort();
    uploadTaskList.value.splice(index, 1);
    if (item.status === 'success') {
        imageList.value.splice(index, 1);
        emit('delItem', { fileItem: imageList.value[index], index });
    }
    uploadIndex.value--;
};

defineExpose({
    uploadTaskList,
    tempFilePaths,
});

watch(
    () => props.preImgList,
    (newVal) => {
        imageList.value = newVal?.map((item) => {
            return {
                url: item.url,
                name: item.name,
                size: item.size,
                status: 'success',
            };
        });
        tempFilePaths.value = newVal?.map((item) => {
            return {
                url: item.url,
                size: item.size,
                status: 'success',
                name: 'file',
            };
        });
        uploadIndex.value = newVal.length;
        uploadTaskList.value = newVal?.map((item) => {
            return {
                imgItem: {
                    url: item.url,
                    size: item.size,
                    status: 'success',
                    name: 'file',
                },
                uploadItem: null,
            };
        });
    },
);
</script>
