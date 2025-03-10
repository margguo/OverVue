<!--
Description:
  Displays Mockup Image upload dropdown as well as minified mockup image once selected
  Functionality includes: importing and clearing mockup image
  -->

<template>
  <div class="home-sidebar drawer-menu">
    <q-list>
      <q-expansion-item expand-separator label="Upload Mockup Image">
        <div class="upload">
          <q-btn
            class="upload-btn"
            color="secondary"
            label="Upload Mockup"
            @click="importMockup"
          />
          <q-btn
            v-if="source !== ''"
            class="upload-btn"
            color="secondary"
            label="Clear Image"
            @click="removeImage"
          />
          <q-btn
            v-else
            disable
            class="upload-btn"
            color="secondary"
            label="Clear Image"
          />
        </div>
        <div class="file-path">
          <q-card>
            <img
              class="img"
              v-if="this.imagePath[this.activeRoute] !== ''"
              :src="'file:///' + this.imagePath[this.activeRoute]"
            />
          </q-card>
        </div>
      </q-expansion-item>
    </q-list>
  </div>
</template>

<script>
import { mapState, mapActions } from "vuex";
import uploadImage from "../../utils/uploadImage.util";
import clearImageDialog from "../../utils/clearImage.util";

export default {
  name: "upload-image",
  data() {
    return {
      files: [],
      source: "",
    };
  },
  computed: {
    ...mapState(["imagePath", "activeRoute"]),
  },
  methods: {
    ...mapActions(["importImage", "clearImage"]),
    // imports mockup image
    // ** Importing mockup image ONLY works in build mode due to path differences
    // In dev mode, error thrown is: "Not allowed to load local resource: PATH "
    importMockup() {
      // A promise gets returned out from uploadImage, imported from uploadImage utils
      const helperPromise = uploadImage();
      helperPromise
        // res contains the selected file path (string)
        .then((res) => {
          if (this.activeRoute !== "") {
            this.importImage({ img: res, route: this.activeRoute });
            if (this.imagePath[this.activeRoute]) {
              this.source = "file:///" + this.imagePath[this.activeRoute];
            }
          }
        })
        .catch((err) => console.log(err));
    },
    // removes mockup image
    removeImage() {
      const responsePromise = clearImageDialog();

      responsePromise
        .then((res) => {
          // res will have format: { response: 0, checkboxChecked: false }
          // res.response will be 0 if user chose 'Yes'
          // res.response will be 1 if user chose 'Cancel'
          if (res.response === 0) {
            this.clearImage({ route: this.activeRoute });
            this.source = this.imagePath[this.activeRoute];
          }
        })
        .catch((err) => {
          console.log(err);
        });
    },

    // imports image on browser
    // currently images aren't able to be stored on browser
    async importMockupBrowser() {
      this.files = this.$refs.myFiles.files[0];
      await this.importImage(this.files.name);
    },
    // removes image on browser
    removeImageBrowser() {
      this.clearImage();
    },
  },
  // watches for changes in state to activeRoute
  watch: {
    // once you change your active route, the mockup image should change as well
    activeRoute: function () {
      if (this.imagePath[this.activeRoute]) {
        // if there is a uploaded image
        this.source = "file:///" + this.imagePath[this.activeRoute];
      } else {
        this.source = "";
      }
    },
  },
};
</script>

<style lang="scss">
.home-sidebar {
  margin: 1rem;
  justify-content: center;
  border-radius: 5px;
  padding: 0px;
  background: $subsecondary;
}

.upload-btn {
  text-transform: capitalize;
  font-size: 12px;
}

.upload {
  margin: 0.5rem;
  display: flex;
  justify-content: space-evenly;
}

.file-path {
  padding-bottom: 1em;
  height: 100%;
  margin: 1rem;
  font-size: 11px;
}

.file-header {
  padding-left: 0.4em;
}

.file-content {
  padding: 0em 1em 1em 1em;
}

.browser-btn {
  width: 90px;
  background: $secondary;
}

.img {
  max-height: 200px;
}
</style>
