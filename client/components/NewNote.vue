<template>
  <div class="new-note">
    <h1 class="title">
      New note
    </h1>
    <h2 class="subtitle">
      {{ subtitle }}
    </h2>
    <div v-if="completed"
         class="finish">
      <div class="field has-addons">
        <p class="control is-expanded">
          <input class="input"
                 type="text"
                 :value="url">
        </p>
        <p class="control">
          <button class="button is-primary"
                  v-clipboard="url">
            <i class='fa fa-clone fa-fw'></i>
            Copy
          </button>
        </p>
      </div>
      Have another secret?
      <a @click="clear">click here to create a new one</a>
    </div>
    <form v-else
          @submit.prevent="send">
      <div class="columns">
        <div class="column is-three-quarters">
          <div class="field">
            <label class="label">Note</label>
            <p class="control">
              <textarea required
                        class="textarea"
                        v-model="note.data"
                        placeholder="Your note"></textarea>
            </p>
          </div>
        </div>
        <div class="column">
          <div class="field">
            <label class="label">Encryption Key</label>
            <div class="field">
              <label class="checkbox">
                <input v-model="generatePassword"
                       type="checkbox"> Generate
                an encryption key for me
              </label>
            </div>
            <div v-if="!generatePassword"
                 class="field">
              <div class="control has-icons-left">
                <input class="input"
                       required=""
                       type="password"
                       placeholder="Encryption key"
                       v-model="userPassword">
                <span class="icon is-small is-left">
                  <i class="fa fa-key"></i>
                </span>
                <p class="help">This key will be prompted for when trying
                  to read the note</p>
              </div>
            </div>
          </div>
          <div class="field">
            <p class="control">
              <button class="button is-primary is-fullwidth"
                      :class="{'is-loading': loading}">Encrypt and store</button>
            </p>
          </div>
        </div>
      </div>
    </form>
  </div>
</template>

<script>
import pbkdf2 from 'pbkdf2'
import aesjs from 'aes-js'
import crypto from 'crypto'
import { promisify } from 'es6-promisify'

const initialState = () => ({
  note: {
    data: null
  },
  loading: false,
  completed: false,
  generatePassword: true,
  userPassword: null,
  generatedPassword: null,
  id: null
})

export default {
  data: initialState,
  methods: {
    /**
     * Sends the encrypted note to the store endpoint
     */
    send() {
      this.loading = true
      this.encrypt(this.note.data)
        .then(body => this.$axios.$post('notes', { body }))
        .then(({ note }) => {
          this.id = note._id
          this.completed = true
          this.loading = false
        })
    },
    /**
     * Clears the state, triggered when the user wants to create another note
     */
    clear() {
      Object.assign(this.$data, initialState())
      this.generateRandomString().then(p => (this.generatedPassword = p))
    },
    /**
     * Used as the password in case the user wants us to generate it for them
     */
    async generateRandomString() {
      const asyncRandomBytes = promisify(crypto.randomBytes)
      const randomBytes = await asyncRandomBytes(16)
      return randomBytes.toString('hex')
    },
    /**
     * Generates the 256bit AES decryption key using the password
     * @returns {Array}
     */
    async generateAesKey() {
      return pbkdf2.pbkdf2Sync(this.password, '~S.uperR4ndOmS4lt1!1', 1, 256 / 8, 'sha512')
    },
    /**
     * AES encrypts a note using a key
     *
     * @param {String} text text to be encrypted
     * @returns {String} encrypted text, in hex
     */
    async encrypt(text) {
      const encryptionKey = await this.generateAesKey()

      const textBytes = aesjs.utils.utf8.toBytes(text)

      /* eslint new-cap: ["error", { "newIsCap": false }] */
      const aesCtr = new aesjs.ModeOfOperation.ctr(encryptionKey)
      const encryptedBytes = aesCtr.encrypt(textBytes)

      return aesjs.utils.hex.fromBytes(encryptedBytes)
    }
  },
  computed: {
    subtitle() {
      return this.completed
        ? 'Note link is ready, go share it with the world!'
        : 'Use the form below to enter your sensitive information and retrieve a URL'
    },
    /**
     * The password used to generate the encryption key, used the generated random string as a password
     * if user chose to not use its own password
     */
    password() {
      return this.generatePassword ? this.generatedPassword : this.userPassword
    },
    /**
     * The shareable note url, only appends the password to the hash parameter if the user
     * did not enter its own password, if the password is not appended to the url it will be prompted for
     * when trying to read the note.
     */
    url() {
      return `${window.origin}/${this.id}${this.generatePassword ? '#' + this.password : ''}`
    }
  },
  /**
   * Generates the random password, ideally this logic would be in the password computed
   * property, but Vue doesn't support async computed values.
   */
  mounted() {
    this.generateRandomString().then(p => (this.generatedPassword = p))
  }
}
</script>
