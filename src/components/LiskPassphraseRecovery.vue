<template>
  <div class="form">
    <h1>Lisk Passphrase Whitespace Checker</h1>
    <p>Check a 12-word Lisk passphrase for extra white space</p>
    <input v-model="passphrase" placeholder="Enter Passphrase"/>
    <br>
    <button v-on:click="lookupPassphrase">Search for a valid variation of the entered passphrase </button>
    <div v-if="active">
      <div> Trying passphrase: </div>
      <pre> {{ active }} </pre>
    </div>
    <div v-if="found">
      <h4> Found a simillar passphrase with {{account.balance / 1e8}} LSK </h4>
      <input v-model="found" readonly placeholder="Enter Passphrase"/>
      <div>
        {{ hint }}
      </div>
    </div>
    <div v-if="error">
      <h4>Error: {{ error }} </h4>
    </div>
  </div>
</template>

<script>
import Lisk from 'lisk-js';


export default {
  name: 'LiskPassphraseRecovery',
  methods: {
    lookupPassphrase() {
      this.found = '';
      this.error = '';
      this.hint = '';
      const passphrase = this.passphrase;

      const words = passphrase.split(' ');
      const validVariations = [];
      [...words, ''].forEach((word, i) => {
        const ws = [...words];
        ws.splice(i, 0, '');
        validVariations.push(ws.join(' '));
      });
      this.resolvePass(validVariations);
    },
    getAccount(passphrase, callback) {
      const { publicKey } = Lisk.crypto.getKeys(passphrase);
      const address = Lisk.crypto.getAddress(publicKey);
      const testnet = !!(new URL(location.toString()).searchParams.get('testnet'));
      Lisk.api({ testnet }).getAccount(address, callback);
    },
    resolvePass(validVariations) {
      const passphrase = validVariations.pop();
      this.active = passphrase;
      this.getAccount(passphrase, (data) => {
        if (data.success) {
          this.found = passphrase;
          this.active = '';
          this.account = data.account;
          const words = passphrase.split(' ');
          const indexOfSpace = words.indexOf('');
          if (indexOfSpace === words.length - 1) {
            this.hint = 'Hint: There is extra space after the last word';
          } else if (words[indexOfSpace + 1]) {
            this.hint = `Hint: There is extra space before word "${words[indexOfSpace + 1]}"`;
          }
        } else if (validVariations.length > 0) {
          const that = this;
          setTimeout(() => {
            that.resolvePass(validVariations);
          }, 100);
        } else {
          this.error = 'No match found';
          this.active = '';
        }
      });
    },
  },
  data: () => ({
    passphrase: '',
    active: '',
    found: '',
    error: '',
    hint: '',
  }),
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

input,
button {
  font-size: 1.2em;
  padding: 8px;
  margin: 8px;
}

input {
  width: 700px;
}

button {
  width: 718px;
}
</style>
