<template>
  <b-modal :active.sync="isActive" :width="640" scroll="keep">
    <div class="box">
      <h3 class="title is-5">Connect your email server</h3>
      <b-field label="Provider">
        <b-select v-model="form.provider">
          <option v-for="p in providers" :key="p.value" :value="p.value">{{ p.label }}</option>
        </b-select>
      </b-field>

      <b-field label="Host">
        <b-input v-model="form.host" :readonly="isPreset"></b-input>
      </b-field>

      <b-field label="Port">
        <b-input type="number" v-model.number="form.port" :readonly="isPreset"></b-input>
      </b-field>

      <b-field label="Username (email)">
        <b-input v-model="form.username"></b-input>
      </b-field>

      <b-field label="Password / API key">
        <b-input v-model="form.password" type="password"></b-input>
      </b-field>

      <b-field label="Encryption">
        <b-select v-model="form.tls">
          <option value="starttls">STARTTLS</option>
          <option value="ssl_tls">SSL/TLS</option>
          <option value="off">Off</option>
        </b-select>
      </b-field>

      <b-field label="Auth protocol">
        <b-select v-model="form.auth">
          <option value="login">LOGIN</option>
          <option value="plain">PLAIN</option>
          <option value="cram">CRAM</option>
          <option value="none">None</option>
        </b-select>
      </b-field>

      <b-field class="has-text-right">
        <b-button type="is-primary" :loading="isSaving" @click="save">Save & Test</b-button>
        <b-button @click="close" class="ml-2">Cancel</b-button>
      </b-field>
    </div>
  </b-modal>
</template>

<script>
import Vue from 'vue';

export default Vue.extend({
  name: 'ConnectEmail',
  props: {
    active: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      isActive: this.active,
      isSaving: false,
      providers: [
        { label: 'Gmail', value: 'gmail', defaults: { host: 'smtp.gmail.com', port: 587, tls: 'starttls', auth: 'login' } },
        { label: 'SendGrid', value: 'sendgrid', defaults: { host: 'smtp.sendgrid.net', port: 587, tls: 'starttls', auth: 'login', username: 'apikey' } },
        { label: 'Mailgun', value: 'mailgun', defaults: { host: 'smtp.mailgun.org', port: 587, tls: 'starttls', auth: 'login' } },
        { label: 'Custom', value: 'custom', defaults: { host: '', port: 587, tls: 'starttls', auth: 'login' } },
      ],
      form: {
        provider: 'gmail',
        host: 'smtp.gmail.com',
        port: 587,
        username: '',
        password: '',
        tls: 'starttls',
        auth: 'login',
      },
    };
  },
  watch: {
    active(v) {
      this.isActive = v;
    },
    isActive(v) {
      this.$emit('update:active', v);
    },
    'form.provider'(val) {
      const preset = this.providers.find((p) => p.value === val);
      Object.assign(this.form, { ...preset.defaults });
    },
  },
  computed: {
    isPreset() {
      return this.form.provider !== 'custom';
    },
  },
  methods: {
    close() {
      this.isActive = false;
    },
    async save() {
      if (!this.form.host || !this.form.port || !this.form.username || !this.form.password) {
        this.$utils.toast('Please fill all required fields', 'is-danger');
        return;
      }
      this.isSaving = true;
      try {
        // fetch existing settings
        const settings = await this.$api.getSettings();
        const smtpArr = [...settings.smtp];
        smtpArr[0] = {
          host: this.form.host.trim(),
          port: this.form.port,
          username: this.form.username.trim(),
          password: this.form.password.trim(),
          auth_protocol: this.form.auth,
          tls: this.form.tls,
          enabled: true,
        };
        await this.$api.updateSettings({ ...settings, smtp: smtpArr });
        this.$utils.toast('SMTP saved! Test it from Settings → SMTP', 'is-success');
        this.isActive = false;
      } catch (err) {
        this.$utils.toast('Failed to save settings', 'is-danger');
      }
      this.isSaving = false;
    },
  },
});
</script>
