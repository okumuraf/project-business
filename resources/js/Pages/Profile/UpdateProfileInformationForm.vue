<template>
  <jet-form-section @submitted="updateProfileInformation">
    <template #title>
      Profile Information
    </template>

    <template #description>
      Update your account's profile information and email address.
    </template>

    <template #form>
      <!-- Profile Photo -->
      <div v-if="$page.jetstream.managesProfilePhotos" class="col-span-6 sm:col-span-4">
        <!-- Profile Photo File Input -->
        <input ref="photo" class="hidden"
               type="file"
               @change="updatePhotoPreview">

        <jet-label for="photo" value="Photo"/>

        <!-- Current Profile Photo -->
        <div v-show="! photoPreview" class="mt-2">
          <img :src="$page.user.profile_photo_url" alt="Current Profile Photo"
               class="rounded-full h-20 w-20 object-cover">
        </div>

        <!-- New Profile Photo Preview -->
        <div v-show="photoPreview" class="mt-2">
                    <span
                        :style="'background-size: cover; background-repeat: no-repeat; background-position: center center; background-image: url(\'' + photoPreview + '\');'"
                        class="block rounded-full w-20 h-20">
                    </span>
        </div>

        <jet-secondary-button class="mt-2 mr-2" type="button" @click.native.prevent="selectNewPhoto">
          Select A New Photo
        </jet-secondary-button>

        <jet-secondary-button v-if="$page.user.profile_photo_path" class="mt-2" type="button"
                              @click.native.prevent="deletePhoto">
          Remove Photo
        </jet-secondary-button>

        <jet-input-error :message="form.error('photo')" class="mt-2"/>
      </div>

      <!-- Name -->
      <div class="col-span-6 sm:col-span-4">
        <jet-label for="name" value="Name"/>
        <jet-input id="name" v-model="form.name" autocomplete="name" class="mt-1 block w-full" type="text"/>
        <jet-input-error :message="form.error('name')" class="mt-2"/>
      </div>

      <!-- Email -->
      <div class="col-span-6 sm:col-span-4">
        <jet-label for="email" value="Email"/>
        <jet-input id="email" v-model="form.email" class="mt-1 block w-full" type="email"/>
        <jet-input-error :message="form.error('email')" class="mt-2"/>
      </div>
    </template>

    <template #actions>
      <jet-action-message :on="form.recentlySuccessful" class="mr-3">
        Saved.
      </jet-action-message>

      <jet-button :class="{ 'opacity-25': form.processing }" :disabled="form.processing">
        Save
      </jet-button>
    </template>
  </jet-form-section>
</template>

<script>
import JetButton from './../../Jetstream/Button'
import JetFormSection from './../../Jetstream/FormSection'
import JetInput from './../../Jetstream/Input'
import JetInputError from './../../Jetstream/InputError'
import JetLabel from './../../Jetstream/Label'
import JetActionMessage from './../../Jetstream/ActionMessage'
import JetSecondaryButton from './../../Jetstream/SecondaryButton'

export default {
  components: {
    JetActionMessage,
    JetButton,
    JetFormSection,
    JetInput,
    JetInputError,
    JetLabel,
    JetSecondaryButton,
  },

  props: ['name', 'email'],

  data() {
    return {
      form: this.$inertia.form({
        '_method': 'PUT',
        name: this.name,
        email: this.email,
        photo: null,
      }, {
        bag: 'updateProfileInformation',
        resetOnSuccess: false,
      }),

      photoPreview: null,
    }
  },

  methods: {
    updateProfileInformation() {
      if (this.$refs.photo) {
        this.form.photo = this.$refs.photo.files[0]
      }

      this.form.post('/user/profile-information', {
        preserveScroll: true
      });
    },

    selectNewPhoto() {
      this.$refs.photo.click();
    },

    updatePhotoPreview() {
      const reader = new FileReader();

      reader.onload = (e) => {
        this.photoPreview = e.target.result;
      };

      reader.readAsDataURL(this.$refs.photo.files[0]);
    },

    deletePhoto() {
      this.$inertia.delete('/user/profile-photo', {
        preserveScroll: true,
      }).then(() => {
        this.photoPreview = null
      });
    },
  },
}
</script>
