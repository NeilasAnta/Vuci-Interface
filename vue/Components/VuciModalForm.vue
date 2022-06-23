<template>
  <vuci-form :uci-config="this.uciConfig">
        <vuci-named-section :name="this.name" v-slot="{ s }" :card="false">
          <vuci-form-item-select :uci-section="s" :label="'Protocol'" name="proto" :options="protos" initial="static" required/>
          <vuci-form-item-input :uci-section="s" :label="'IP address'" rules="ipaddr" name="address" depend="proto == 'static'"  required/>
          <vuci-form-item-input :uci-section="s" :label="'Netmask'" rules="netmask4" name="netmask" depend="proto == 'static'" required/>
          <vuci-form-item-input :uci-section="s" :label="'Gateway'" rules="ipaddr" name="gateway" depend="proto == 'static'" />
          <vuci-form-item-list :uci-section="s" :label="'DNS servers'" rules="ipaddr" name="dns" depend="proto == 'static'" />
          <vuci-form-item-switch :uci-section="s" :label="'DHCP'" name="enable" initial true-value="1" false-value="0"/>
        </vuci-named-section>
      </vuci-form>
</template>

<script>
export default {
  name: 'VuciModalForm',
  props: {
    uciConfig: String,
    name: String
  },
  data () {
    return {
      protos: ['dhpc', 'static']
    }
  }
}
</script>
