<template>
  <div class="form">
    <h1>Lisk Passphrase Recovery</h1>
    <p>Recover a 12-word Lisk passphrase with exactly one incorrect word</p>
    <input v-model="passphrase" placeholder="Enter Passphrase"/>
    <br>
    <button v-on:click="lookupPassphrase">Search for a valid variation of the entered passphrase </button>
    <div v-if="active">
      <div> Trying passphrase: </div>
      <div> {{ active }} </div>
    </div>
    <div v-if="found">
      <h4> Found a passphrase with some LSK: </h4>
      <h3>{{ found }} </h3>
      <div>
        <a :href="'https://explorer.lisk.io/address/' + address">
          {{ address }}
        </a>
      </div>
    </div>
    <div v-if="error">
      <h4>Error: {{ error }} </h4>
    </div>
  </div>
</template>

<script>
import bip39 from 'bip39';
import Lisk from 'lisk-js';


export default {
  name: 'LiskPassphraseRecovery',
  methods: {
    lookupPassphrase() {
      const passphrase = this.passphrase;

      const words = passphrase.split(' ');
      const validVariations = [];
      words.forEach((word, i) => {
        bip39.wordlists.EN.forEach((dictWord) => {
          const ws = [...words];
          ws[i] = dictWord;
          const pass = ws.join(' ');
          const isValid = bip39.validateMnemonic(pass);
          if (isValid) {
            validVariations.push(pass);
          }
        });
      });
      this.resolvePass(validVariations);
    },
    resolvePass(validVariations) {
      const passphrase = validVariations.pop();
      this.active = passphrase;
      const { publicKey } = Lisk.crypto.getKeys(passphrase);
      const address = Lisk.crypto.getAddress(publicKey);
      Lisk.api({ testnet: false }).getAccount(address, (data) => {
        if (data.success) {
          this.found = passphrase;
          this.active = '';
          this.address = address;
        } else if (validVariations.length > 0) {
          this.resolvePass(validVariations);
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
