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
            <image
                mode="aspectFit"
                src="data:image/png;base64,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"
            />
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
