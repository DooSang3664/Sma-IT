<template>
    <v-container class="col-6 centerContent">
        <p class="centerText registerTitle">고객 등록</p>
        <div class="col-6 centerContent">
            <div>
                <v-spacer></v-spacer>
                <div class="pb-5">
                    <img v-if="imageUrl == ''" :src="imageUrl" alt="" />
                    <img v-else :src="imageUrl" class="profileImg" alt="profileImg" />
                </div>
                <v-row>
                    <v-spacer></v-spacer>
                    <p class="profileName">{{ imageName }}</p>

                    <input ref="imageInput" type="file" hidden @change="onChangeImages" />
                    <v-btn type="button" @click="onClickImageUpload">이미지 업로드</v-btn>
                </v-row>
                <v-spacer></v-spacer>
            </div>

            <v-row>
                <v-text-field
                    v-model="member.name"
                    label="name"
                    type="string"
                    dark
                    class="nameField"
                    style="font-size:12px;"
                ></v-text-field>
                <v-spacer></v-spacer>
                <v-text-field
                    v-model="member.age"
                    label="나이"
                    type="number"
                    dark
                    class="ageField"
                    style="font-size:12px;"
                ></v-text-field>
            </v-row>
            <v-text-field
                v-model="getMember.card_num"
                label="카드번호"
                placeholder=" '-'를 빼고, 숫자만 16글자 입력해주세요."
                type="string"
                dark
                style="font-size:12px;"
            ></v-text-field>
            <v-text-field
                v-model="member.interests"
                label="관심분야"
                type="string"
                dark
                style="font-size:12px;"
            >
                ></v-text-field
            >
            <v-text-field
                v-model="member.requirements"
                label="요구사항"
                type="string"
                dark
                style="font-size:12px;"
            ></v-text-field>
            <v-row>
                <v-spacer></v-spacer>
                <button class="resetBtn" @click="resetInformation">초기화</button>

                <button class="registBtn" @click="registInformation">등록</button>
            </v-row>
        </div>
    </v-container>
</template>

<script>
import http from '../api/axios';
import swal from 'sweetalert';
export default {
    name: 'customerRegister',
    data() {
        return {
            imageName: this.imageName,
            imageFile: '',
            imageUrl: '',
            res: '',
            member: {
                name: '',
                age: '',
                interests: '',
                requirements: '',
                image: '',
            },

            getMember: {
                uuid: '',
                card_num: 0,
                card_name: '',
            },
        };
    },
    methods: {
        onClickImageUpload() {
            this.$refs.imageInput.click();
        },
        onChangeImages(e) {
            console.log(e.target.files);
            const file = e.target.files[0];
            this.imageFile = file;
            this.imageUrl = URL.createObjectURL(file);
            this.imageName = file.name;
        },
        resetInformation() {
            this.member.name = '';
            this.member.age = '';
            this.getMember.card_num = '';
            this.member.interests = '';
            this.member.requirements = '';
        },
        registInformation() {
            // 이미지를 넣지 않았을 때.
            if (this.imageFile == '' || this.imageName == '') {
                swal('이미지를 넣어 주세요!', {
                    icon: 'error',
                });
            }
            // 이름을 넣지 않았을 때.
            else if (this.member.name == '') {
                swal('이름을 입력해주세요!', {
                    icon: 'error',
                });
            }
            // 나이를 넣지 않았을 때
            else if (this.member.age == '') {
                swal('나이를 입력해주세요!', {
                    icon: 'error',
                });
            }
            // 나이가 0보다 작거나 150 이상일 때
            else if (this.member.age < 0 || this.member.age > 150) {
                swal('잘못된 정보 입니다!', {
                    icon: 'error',
                });
            }
            // 카드 정보를 입력하지 않았을 때
            else if (this.member.cardNumber == '') {
                swal('카드 정보를 입력해주세요!', {
                    icon: 'error',
                });
            }
            // 카드 정보가 16자보다 작을 때
            else if (this.getMember.card_num.length < 16) {
                swal('잘못된 카드 정보 입니다!', {
                    icon: 'error',
                });
            }
            // 모두 입력한 경우.
            else {
                if (this.member.interests == '') this.member.interests = '없음';
                if (this.member.requirements == '') this.member.requirements = '없음';

                let formData = new FormData();
                formData.append('file', this.imageFile);
                // 멤버 등록
                http.post('/member/', this.member)
                    .then((response) => {
                        this.imageName = response.data.image;
                        this.getMember.uuid = response.data.uuid;
                        this.getMember.card_name = response.data.name;

                        http.post('/pay/register', this.getMember)
                            .then(() => {
                                console.log('카드등록성공');
                            })
                            .catch(() => {
                                console.log('카드등록실패');
                            });
                        http.post(`/member/image/${this.imageName}`, formData)
                            .then(() => {
                                swal('등록 성공!', {
                                    icon: 'success',
                                });
                                this.$router.push({ name: 'Cam' });
                            })
                            .catch(() => {
                                swal('등록 실패!', {
                                    icon: 'error',
                                });
                            });
                    })
                    .catch((response) => {
                        swal('등록 실패!', {
                            icon: 'error',
                        });
                        console.log(response);
                    });
            }
        },
    },
};
</script>

<style scoped>
@import '../assets/css/customerRegister.css';
</style>
