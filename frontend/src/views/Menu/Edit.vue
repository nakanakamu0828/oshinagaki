<template>
  <div>
    <div class="flex justify-between items-center">
      <div class="w-1/3">
        <return-button to="/menus" />
      </div>
      <page-title title="メニュー編集" subtitle="メニューの編集ページです" icon="event" />
      <div class="w-1/3 text-right"></div>
    </div>
    <div class="lg:w-2/3 mx-auto my-5">
      <menu-form
        v-show="menu"
        :menu="menu"
        :wallpapers="wallpapers"
        :fonts="fonts"
        :errors="errors"
        @save-menu="updateMenu"
      />
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import { Menu, Wallpaper, Font, ServerErrors } from '@/store/types';
import ReturnButton from '@/components/Atoms/ReturnButton.vue';
import PageTitle from '@/components/Molecules/PageTitle.vue';
import MenuForm from '@/components/Organisms/MenuForm/Index.vue';
import axios from 'axios';
import dayjs from 'dayjs';

@Component({
  components: {
    ReturnButton,
    PageTitle,
    MenuForm,
  },
})
export default class MenuEditView extends Vue {
  private menu: Menu | null = null;
  private wallpapers: Wallpaper[] = [];
  private fonts: Font[] = [];
  private errors: ServerErrors = {};

  public created() {
    Promise.all([
      axios.get('/api/wallpapers'),
      axios.get('/api/fonts'),
      axios.get(`/api/menu/${this.$route.params.id}`)
    ]).then(([
      wallpapersResponse,
      fontsResponse,
      menuResponse,
    ]) => {
      this.wallpapers = wallpapersResponse.data;
      this.fonts = fontsResponse.data;
      const menu = menuResponse.data;
      menu.date = dayjs(menu.date).format('YYYY-MM-DD');
      this.menu = menu;
    });
  }

  private updateMenu(menu: Menu) {
    axios.post(`/api/menu/${this.$route.params.id}/update`, menu)
      .then(response => {
        this.$router.push('/menus');
      })
      .catch(err => {
        const response = err.response;
        this.errors = response.data.errors as ServerErrors;
        console.log(this.errors)
      })
    ;
  }
}
</script>
