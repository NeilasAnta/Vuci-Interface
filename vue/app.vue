
<template>
  <div>
    <vuci-form uci-config="task" ref="childComponentRef">
      <vuci-typed-section type="interface" :columns="columns">
          <template #name="{ s }">
            <vuci-form-item-dummy :uci-section="s"  name="name"/>
          </template>
          <template #address="{ s }">
            <vuci-form-item-dummy :uci-section="s"  name="address"/>
          </template>
          <template #netmask="{ s }">
            <vuci-form-item-dummy :uci-section="s"  name="netmask"/>
          </template>
          <template #action="{ s }">
            <a-button type="primary" size="small" style="margin-right: 5px" @click="edit(s['.name'])">{{ $t('Edit') }}</a-button>
            <a-popconfirm @confirm="del(s['.name'])" placement="left" :title="'Really delete this interface?'">
              <a-button type="danger" size="small">{{ $t('Delete') }}</a-button>
            </a-popconfirm>
          </template>
      </vuci-typed-section>
      <template slot="footer"><div></div></template>
    </vuci-form>
    <a-button type="primary" size="small" style="margin-top: 10px" @click="handleAdd">+ {{ $t('interfaces.Add interface') }}</a-button>
    <a-modal v-model="editModal" :width="800">
      <vuci-modal-form uciConfig="task" :name="editorIface" v-if="editModal"></vuci-modal-form>
          <template #footer><div/></template>
    </a-modal>
  
  </div>
  
</template>

<script>
import VuciModalForm from './Components/VuciModalForm.vue';
export default {
  components: { VuciModalForm },
  data () {
    return {
      interfaces: [],
      columns: [
        { name: 'name', label: 'Interface name',},
        { name: 'address', label: 'IP address' },
        { name: 'netmask', label: 'Netmask' },
        { name: 'action', label: 'Action' },
      ],
      editorIface: '',
      editModal: false,
      switchProto: false,
    }
  },
  methods:{
    load () {
      this.uciConfig.load().then(() => {
        this.interfaces = this.$uciConfig.getInterfaces()
      })
    },
    edit (iface) {
      this.proto = this.$uci.get('task', iface)
      this.editorIface = iface
      this.editModal = true
    },
    add (name) {
      this.$spin()
      const sid = this.$uci.add('task', 'interface')
      this.$uci.set('task', sid, 'name', name)
      this.$uci.set('task', sid, 'proto', 'static')
      this.$uci.set('task', sid, 'dhcp', '1')
      this.$uci.save().then(() => {
        this.$uci.apply().then(() => {
          this.updateField()
          this.$spin(false)
        })
      })
    },
    del (name) {
      this.$spin()
      this.$uci.del('task', name)
      this.$uci.save().then(() => {
        this.$uci.apply().then(() => {
          this.updateField()
          this.$spin(false) 
        })
      })
      this.$uci.reset()
    },
    updateField()
    {
      this.$refs.childComponentRef._data.loaded = false;
      this.$refs.childComponentRef.load()
    },
    handleAdd () {
      this.$prompt({
        title: this.$t('interfaces.Add interface'),
        placeholder: this.$t('Please input a name')
      }).then(name => {
        if (!name) return

        if (name.match(/^[a-zA-Z0-9_]+$/) === null) {
          this.$message.error(this.$t('validator.uci'))
          return
        }

        this.add(name)
      }).catch(() => {})
    }
  },
  watch:{
    editModal(newEditModal, oldEditModal)
    {
      if(oldEditModal === true && newEditModal === false)
      {
        this.updateField()
      }
    }
  }
}
</script>
