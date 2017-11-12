<template>
  <div class="form">
    <h1>Lisk Passphrase Recovery</h1>
    <p>Recover a 12-word Lisk passphrase with exactly one incorrect word</p>
    <input v-model="passphrase" placeholder="Enter Passphrase"/>
    <br>
    <button v-on:click="lookupPassphrase">Search for a valid variation of the entered passphrase </button>
    <div> {{ active }} </div>
  </div>
</template>

<script>
import bip39 from 'bip39';
import Lisk from 'lisk-js';

const resolvePass = (validVariations) => {
  const passphrase = validVariations.pop();
  const { publicKey } = Lisk.crypto.getKeys(passphrase);
  const address = Lisk.crypto.getAddress(publicKey);
  Lisk.api({ testnet: false }).getAccount(address, (data) => {
    if (data.success) {
      alert(`Found a passphrase with some LSK on it: ${passphrase}`); // eslint-disable-line no-alert
    } else if (validVariations.length > 0) {
      resolvePass(validVariations);
    } else {
      alert('No match found'); // eslint-disable-line no-alert
    }
  });
};

export default {
  name: 'LiskPassphraseRecovery',
  methods: {
    lookupPassphrase: () => {
      const passphrase = this.a.data().passphrase;
      this.a.data().result = this.a.data().passphrase;

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
      resolvePass(validVariations);
    },
  },
  data: () => ({
    passphrase: '',
    active: '',
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
