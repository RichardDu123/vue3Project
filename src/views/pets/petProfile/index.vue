<template>
  <div>
    <van-nav-bar
      left-arrow
      @click-left="onClickLeft"
      title="Pet Page"
      class="myNav"
      fixed
    />
    <div class="petBasicInfo">
      <!--      <van-image round :src="petContent.petAvatar" class="avatar" fit="cover" />-->
      <!--      <div class="nameWrapper">-->
      <!--        <van-cell-group class="name" inset>-->
      <!--          <van-field v-model="petContent.petName" placeholder="Meow" readonly />-->
      <!--        </van-cell-group>-->
      <!--      </div>-->
      <div class="pet-avatar-line">
        <van-image
          round
          :src="avatarUrl"
          class="pet-avatar"
          fit="cover"
          @click="avatarClick"
        />
        <van-uploader
          accept="image/png, image/jpeg"
          v-model="fileList"
          ref="target"
          v-show="false"
        ></van-uploader>
        <div class="pet-name">
          <p style="font-size: large">{{ petContent.petName }}</p>
        </div>
        <van-button
          hairline
          icon="edit"
          type="primary"
          @click="handleEditClick"
          class="pet-edit-button"
          size="small"
        >
          {{ status }}
        </van-button>
      </div>
      <div class="pet-intro">
        <van-cell-group inset>
          <van-field
            v-model="petContent.description"
            rows="2"
            autosize
            type="textarea"
            maxlength="100"
            placeholder="Add text"
            readonly
          />
        </van-cell-group>
      </div>
      <div class="change-info-container" v-if="status !== 'Edit'">
        <van-divider
          :style="{
            padding: '0 10px',
            color: 'white',
            borderColor: 'white',
          }"
        >
          Change your profile below
        </van-divider>
        <div>
          <van-cell-group inset>
            <van-field
              v-model="petContent.petName"
              label="Pet Name: "
              placeholder="Please enter your pet name"
            />
            <van-field
              v-model="petContent.description"
              rows="2"
              autosize
              type="textarea"
              maxlength="50"
              placeholder="Description of yourself"
            />
          </van-cell-group>
        </div>
      </div>
    </div>
    <van-divider :style="{ padding: '0 10px' }"> Pet Gallery </van-divider>
    <van-swipe :autoplay="3000" lazy-render class="imgSwipe">
      <van-swipe-item v-for="image in petContent.petImgList" :key="image">
        <img :src="image" @click="handlePreview" />
      </van-swipe-item>
    </van-swipe>
  </div>
</template>

<script lang="ts">
import { useRoute, useRouter } from 'vue-router'
import { computed, reactive, ref } from 'vue'
import { ImagePreview, Notify } from 'vant'
import 'vant/es/image-preview/style'
import { getPetById } from '@/api/pet'
import { updateUserInfo } from '@/api/user'
import { usePetStore } from '@/stores/pet'

export default {
  setup() {
    // route back
    const route = useRouter()
    const onClickLeft = () => {
      route.back()
    }
    const petContent = reactive({
      petAvatar: '',
      petName: '',
      description: '',
      petImgList: [],
    })

    // get pet info
    const currRoute = useRoute()
    getPetById(currRoute.params.petId as string, {}).then((value) => {
      const { data } = value
      petContent.petAvatar = data.petImageAddress
      petContent.petName = data.petName
      petContent.description = data.petDescription
      petContent.petImgList = data.petImageList
    })
    const handlePreview = () => {
      ImagePreview(petContent.petImgList)
    }

    // update pet info
    const petStore = usePetStore()
    const status = computed(() => (petStore.isDeleteShow ? 'Save' : 'Edit'))
    console.log(status.value)

    const target = ref<any>(null)
    const fileList = ref<any>([])
    const avatarClick = () => {
      console.log(status.value)
      if (status.value === 'Save') {
        target.value.chooseFile()
      }
    }
    const defaultUrl = ref('')
    const avatarUrl = computed(() => {
      return !fileList.value.length
        ? defaultUrl.value
        : fileList.value[fileList.value.length - 1].content
    })

    const petDescription = ref('')
    const petName = ref('')
    const handleEditClick = () => {
      if (status.value === 'Edit') {
        petStore.isDeleteShow = true
      } else {
        petStore.isDeleteShow = false
        updateUserInfo({
          userImageAddress: avatarUrl.value,
          nickName: petName.value,
          description: petDescription.value,
        }).then((value) => {
          if (value.message === 'Success') {
            Notify({ type: 'success', message: 'Successfully update.' })
          }
        })
      }
    }

    return {
      target,
      fileList,
      petStore,
      status,
      petContent,
      petDescription,
      petName,
      route,
      defaultUrl,
      avatarUrl,
      avatarClick,
      handleEditClick,
      onClickLeft,
      handlePreview,
    }
  },
}
</script>

<style scoped lang="less">
.petBasicInfo {
  padding-top: 50px;
  justify-content: space-between;
  align-items: center;
  background-image: url('@/assets/images/pet_background.jpg');
  background-size: 100% 100%;
  padding-bottom: 20px;
  .pet-avatar-line {
    display: flex;
    .pet-avatar {
      margin: 20px 20px;
      width: 110px;
      height: 110px;
      border: 2px solid white;
    }
    .pet-name {
      color: white;
      width: 110px;
      margin: auto;
    }
    .pet-edit-button {
      width: 80px;
      margin: auto;
    }
    .pet-intro {
      margin-bottom: 20px;
    }
  }
}
</style>