<template>
  <q-page class="page-app">
    <q-card
      square
      class="icard"
    >
      <q-bar class="entry-caption">
        {{ pagetitle }}
        <q-space />
        <q-input
          v-model="filter"
          dense
          outline
          debounce="300"
          label-color="white"
          borderless
          placeholder="Pencarian"
          input-class="field-filter"
        >
          <template v-slot:append>
            <q-icon
              v-if="filter === ''"
              name="search"
              size="xs"
              color="white"
            />
            <q-icon
              v-else
              name="clear"
              class="cursor-pointer"
              size="xs"
              color="white"
              @click="filter = ''"
            />
          </template>
        </q-input>
      </q-bar>
      <q-table
        dense
        square
        :rows="data"
        :columns="columns"
        no-data-label="data kosong"
        no-results-label="data yang cari tidak ditemukan"
        row-key="sysid"
        :filter="filter"
        separator="cell"
        selection="single"
        v-model:selected="selected"
        v-model:pagination="pagination"
        binary-state-sort
        @request="onRequest"
        :loading="loading"
        virtual-scroll
        table-class="fit-table-ui"
      >
        <template v-slot:loading>
          <q-inner-loading showing>
            <q-spinner-ball
              size="75px"
              color="red-10"
            />
          </q-inner-loading>
        </template>

        <template v-slot:header="props">
          <q-tr :props="props">
            <q-th
              v-for="col in props.cols"
              :key="col.name"
              :props="props"
            >
              {{ col.label }}
            </q-th>
          </q-tr>
        </template>
        <template v-slot:body="props">
          <q-tr
            :props="props"
            @click="props.selected = !props.selected"
          >
            <q-td
              v-for="col in props.cols"
              :key="col.name"
              :props="props"
            >
              <div class="grid-data">
                <div v-if="col.name === 'action'">
                  <q-icon
                    v-for="(btn, index) in btns"
                    v-show="btn.is_allowed && btn.is_show"
                    :key="index"
                    no-caps
                    flat
                    class="q-mr-sm"
                    :name="btn.icon"
                    size="xs"
                    color="green-10"
                    @click="runMethod(btn.onclick, props.row.uuid_rec)"
                  >
                    <q-tooltip content-class="tooltips-app">
                      {{ btn.tooltips }}
                    </q-tooltip>
                  </q-icon>
                </div>
                <div v-else-if="col.name === 'is_active'">
                  <q-toggle
                    v-model="props.row.is_active"
                    dense
                    disable
                  />
                </div>
                <div v-else-if="col.name === 'created_date'">
                  {{ $INDDateTime(props.row.create_date) }}
                </div>
                <div v-else-if="col.name === 'updated_date'">
                  {{ $INDDateTime(props.row.update_date) }}
                </div>
                <div v-else>
                  {{ col.value }}
                </div>
              </div>
            </q-td>
          </q-tr>
        </template>
      </q-table>
    </q-card>

    <!-- place QPageSticky at end of page -->
    <q-page-sticky
      expand
      position="top"
    >
      <q-toolbar class="main-toolbar">
        <q-btn
          v-for="(btn, index) in btns"
          :key="index"
          dense
          no-caps
          flat
          v-show="btn.is_allowed"
          class="btn-toolbar q-mr-xs"
          :icon="btn.icon"
          :label="btn.caption"
          @click="runMethod(btn.onclick)"
        >
          <q-tooltip content-class="tooltips-app">
            {{ btn.tooltips }}
          </q-tooltip>
        </q-btn>
      </q-toolbar>
    </q-page-sticky>

    <!-- Dialog UI Interface-->
    <q-dialog
      v-model="dataevent"
      persistent
      transition-show="flip-down"
      transition-hide="flip-up"
    >
      <q-card
        class="icard"
        square
      >
        <q-bar class="entry-caption">
          {{ title }}
          <q-space />
          <q-btn
            v-close-popup
            dense
            flat
            rounded
            icon="close"
            size="sm"
          >
            <q-tooltip>Tutup</q-tooltip>
          </q-btn>
        </q-bar>

        <q-card-section class="q-gutter-sm">
          <div class="row items-center q-col-gutter-sm q-mb-sm">
            <div class="col-3">
              <q-input
                v-model="edit.dept_code"
                dense
                outlined
                square
                label="Kode Klinik"
                stack-label
              />
            </div>
            <div class="col-6">
              <q-input
                v-model="edit.dept_name"
                dense
                outlined
                square
                label="Nama Klinik"
                stack-label
              />
            </div>
            <div class="col-3">
              <q-input
                v-model="edit.sort_name"
                dense
                outlined
                square
                label="Singkatan"
                stack-label
              />
            </div>
          </div>
          <div class="row items-start q-col-gutter-sm q-mb-sm">
            <div class="col-12">
              <q-select
                v-model="edit.is_executive"
                dense
                outlined
                square
                label="Jenis Klinik"
                stack-label
                :options="[
                  { value: false, label: 'Non Executive' },
                  { value: true, label: 'Executive' }
                ]"
                option-value="value"
                option-label="label"
                emit-value
                map-options
              />
            </div>
          </div>
          <div class="row items-start q-col-gutter-sm q-mb-sm">
            <div class="col-12">
              <q-input
                v-model="edit.wh_medical_name"
                dense
                outlined
                square
                label="Lokasi Obat"
                stack-label
                readonly
              >
                <template v-slot:append>
                  <q-icon
                    name="search"
                    color="green-10"
                    size="sx"
                    @click="open_warehouse('MEDICAL', 'wh_medical')"
                  />
                </template>
              </q-input>
            </div>
          </div>
          <div class="row items-start q-col-gutter-sm q-mb-sm">
            <div class="col-12">
              <q-input
                v-model="edit.wh_general_name"
                dense
                outlined
                square
                label="Lokasi barang umum"
                stack-label
                readonly
              >
                <template v-slot:append>
                  <q-icon
                    name="search"
                    color="green-10"
                    size="sx"
                    @click="open_warehouse('GENERAL', 'wh_general')"
                  />
                </template>
              </q-input>
            </div>
          </div>
          <div class="row items-start q-col-gutter-sm q-mb-sm">
            <div class="col-12">
              <q-input
                v-model="edit.wh_pharmacy_name"
                dense
                outlined
                square
                label="Unit Farmasi"
                stack-label
                readonly
              >
                <template v-slot:append>
                  <q-icon
                    name="search"
                    color="green-10"
                    size="sx"
                    @click="dlgPharmacy = true"
                  />
                </template>
              </q-input>
            </div>
          </div>
          <div class="row items-start q-col-gutter-sm q-mb-sm">
            <div class="col-12">
              <q-input
                v-model="edit.price_class_name"
                dense
                outlined
                square
                label="Acuan Kelas Tarif Jasa"
                stack-label
                readonly
              >
                <template v-slot:append>
                  <q-icon
                    name="search"
                    color="green-10"
                    size="sx"
                    @click="dlgPriceClass = true"
                  />
                </template>
              </q-input>
            </div>
          </div>
        </q-card-section>
        <q-separator />
        <q-card-section
          class="dialog-action q-pa-sm"
          align="right"
        >
          <q-btn
            class="q-mr-sm"
            icon="save"
            label="Simpan"
            flat
            no-caps
            @click="save_data()"
          >
            <template v-slot:loading>
              <q-spinner class="on-left" />
              Proses
            </template>
          </q-btn>
          <q-btn
            icon="undo"
            label="Batal"
            no-caps
            flat
            v-close-popup
          />
        </q-card-section>
      </q-card>
    </q-dialog>

    <warehouse
      v-if="dlgWarehouse"
      :show="dlgWarehouse"
      :warehouse_group="wh_group"
      @CloseData="getWarehouse"
    />
    <department
      v-if="dlgPharmacy"
      :show="dlgPharmacy"
      enumtype="PHARMACY"
      @CloseData="getPharmacy"
    />
    <priceclass
      v-if="dlgPriceClass"
      :show="dlgPriceClass"
      enumtype="SERVICE"
      @CloseData="getPriceClass"
    />
  </q-page>
</template>

<script>
import warehouse from 'components/master/Warehouse.vue'
import department from 'components/master/Department.vue'
import priceclass from 'components/master/PriceClass.vue'
import { defineComponent, ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { useStore } from 'vuex'
import { useQuasar } from 'quasar'

export default defineComponent({
  name: 'InPatientService',
  components: { warehouse, department, priceclass },
  setup() {
    const $q = useQuasar()
    const $store = useStore()
    const $router = useRouter()

    const dlgWarehouse = ref(false)
    const dlgPharmacy = ref(false)
    const dlgPriceClass = ref(false)
    const wh_group = ref('MEDICAL')
    const field = ref('')

    const edit = ref({})
    const dataevent = ref(false)
    const title = ref('Tambah Data')
    const filter = ref('')
    const loading = ref(false)

    const pagination = ref({
      sortBy: 'sysid',
      descending: false,
      page: 1,
      rowsPerPage: 20,
      rowsNumber: 20
    })
    const data = ref([])
    const selected = ref([])
    const columns = ref([])

    const pagetitle = ref('')
    const api_url = ref({})
    const btns = ref([])
    const access = ref({})

    async function onRequest(props) {
      let { page, rowsPerPage, rowsNumber, sortBy, descending } =
        props.pagination
      let filter = props.filter

      let fetchCount = rowsPerPage === 0 ? rowsNumber : rowsPerPage
      loading.value = true
      data.value = []
      let paylaod = {
        page: page,
        limit: fetchCount,
        filter: filter,
        sortBy: sortBy,
        descending: descending,
        group_name: 'INPATIENT',
        url: api_url.value.retrieve
      }
      let respon = await $store.dispatch('master/GET_DATA', paylaod)
      data.value = respon.data
      pagination.value = {
        rowsNumber: respon.total,
        page: respon.current_page,
        rowsPerPage: respon.per_page,
        sortBy: sortBy,
        descending: descending
      }
      loading.value = false
    }

    async function add_event() {
      dataevent.value = true
      title.value = 'Tambah Data'
      edit.value = {
        sysid: -1,
        dept_code: '',
        dept_name: '',
        sort_name: '',
        dept_group: 'INPATIENT',
        wh_medical: -1,
        wh_general: -1,
        wh_pharmacy: -1,
        wh_medical_name: '',
        wh_general_name: '',
        wh_pharmacy_name: '',
        is_executive: false,
        price_class: -1,
        price_class_name: '',
        is_active: true,
        uuid_rec: ''
      }
    }

    async function edit_event(uuid = '') {
      if (selected.value.length <= 0 && uuid === '') {
        return 0
      }

      uuid = uuid !== '' ? uuid : selected.value[0].uuid_rec
      let payload = {
        url: api_url.value.edit,
        uuid_rec: uuid,
        progress: true
      }
      let respon = await $store.dispatch('master/GET_DATA', payload)
      if (!(typeof respon === 'undefined')) {
        title.value = 'Ubah Data'
        dataevent.value = true
        edit.value = respon
        console.info(JSON.stringify(respon))
      }
    }

    async function delete_event(uuid = '') {
      if (selected.value.length <= 0 && uuid === '') {
        return 0
      }
      $q.dialog({
        title: 'Konfirmasi',
        message: 'Apakah data ini akan di hapus ?',
        cancel: true,
        persistent: true
      }).onOk(() => {
        let payload = {
          uuid_rec: uuid,
          url: api_url.value.delete,
          progress: true
        }

        $store.dispatch('master/DELETE_DATA', payload).then((respon) => {
          let msg = respon.data
          if (respon.success) {
            dataevent.value = false
            loaddata()
          }
          $q.notify({
            color: respon.success ? 'positive' : 'negative',
            textcolor: 'white',
            message: msg,
            position: 'top',
            timeout: 2000
          })
        })
      })
    }

    async function save_data() {
      let payload = {
        data: edit.value,
        url: api_url.value.save,
        progress: true
      }
      let respon = await $store.dispatch('master/POST_DATA', payload)
      if (!(typeof respon === 'undefined')) {
        let msg = respon.data
        if (respon.success) {
          dataevent.value = false
          loaddata()
        }
        $q.notify({
          color: respon.success ? 'positive' : 'negative',
          textcolor: 'white',
          message: msg,
          position: 'top',
          timeout: 2000
        })
      }
    }

    async function loaddata() {
      selected.value = []
      onRequest({
        pagination: pagination.value,
        filter: filter.value
      })
    }

    function runMethod(method, uuid = '') {
      this[method](uuid)
    }

    function open_warehouse(group_name, field_name) {
      wh_group.value = group_name
      field.value = field_name
      dlgWarehouse.value = true
    }

    function getWarehouse(closed, data) {
      dlgWarehouse.value = closed
      if (typeof data.sysid == 'undefined') {
        return 0
      }
      if (field.value === 'wh_medical') {
        edit.value.wh_medical = data.sysid
        edit.value.wh_medical_name =
          data.location_code + ' - ' + data.location_name
      } else {
        edit.value.wh_general = data.sysid
        edit.value.wh_general_name =
          data.location_code + ' - ' + data.location_name
      }
    }

    function getPharmacy(closed, data) {
      dlgPharmacy.value = closed
      if (typeof data.sysid == 'undefined') {
        return 0
      }
      edit.value.wh_pharmacy = data.sysid
      edit.value.wh_pharmacy_name = data.dept_code + ' - ' + data.dept_name
    }

    function getPriceClass(closed, data) {
      dlgPriceClass.value = closed
      if (typeof data.sysid == 'undefined') {
        return 0
      }
      edit.value.price_class = data.sysid
      edit.value.price_class_name = data.price_code + ' - ' + data.descriptions
    }

    onMounted(async () => {
      let property = await $store.dispatch(
        'home/GET_PAGEPROPERTY',
        $router.currentRoute.value.fullPath
      )
      columns.value = property.columns
      pagetitle.value = property.title
      api_url.value = property.url
      btns.value = property.btn
      access.value = property.access
      loaddata()
    })

    return {
      data,
      edit,
      dataevent,
      title,
      filter,
      pagination,
      selected,
      columns,
      pagetitle,
      api_url,
      btns,
      access,
      dlgWarehouse,
      dlgPharmacy,
      dlgPriceClass,
      wh_group,
      loading,
      field,
      runMethod,
      onRequest,
      add_event,
      edit_event,
      delete_event,
      loaddata,
      save_data,
      open_warehouse,
      getWarehouse,
      getPharmacy,
      getPriceClass
    }
  }
})
</script>
