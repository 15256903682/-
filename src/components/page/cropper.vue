<template>
    <div>
        <shn-upload-img
            :cropper="true"
            @change="handelChange"
            addText="裁剪图片/上传"
            cropType="base64"
            v-model="cropper.list"
            :width="300"
            :height="300"
            :fixedNumber="[1.4,2]"
            :max="1"
        />
    </div>
</template>
<script>
export default {
    name: 'cropper',
    data() {
        return {
            cropper: {
                list: []
            }
        };
    },
    methods: {
        handelChange(data) {
            this.fnAvater(data);
        },
        fnAvater(key) {
            this.$axios({
                method: 'post',
                url: 'http://192.168.1.58:8082/ymzs/api/curriculum/addImage',
                params: {
                    img: key
                }
            })
                .then((res) => {
                    console.log(res);
                    if (res.data.code == 0) {
                        alert('头像修改成功!');
                    }
                })
                .catch((error) => {
                    console.log(error);
                });
        }
    }
};
</script>  
