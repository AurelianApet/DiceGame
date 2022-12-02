<template>
  <div class="header">
    <div class="logo" @click="activatePage('board')">
      <img src="imgs/logo.png" />
    </div>
    <div class="actions">
      <button @click="activatePage('pool')">
        <img src="imgs/icons/wallet.svg" />
        <span>POOL</span>
      </button>

      <button @click="activatePage('swap')">
        <img src="imgs/icons/wallet.svg" />
        <span>SWAP</span>
      </button>

      <button>
        <img src="imgs/icons/wallet.svg" />
        <span>{{tokenBalance}} DICE</span>
      </button>

      <button @click="toggleWallet()" class="wallet">{{getWalletAdrAbbr}}</button>
    </div>
  </div>
</template>

<script>
import { mapMutations, mapGetters } from 'vuex'
import Web3 from 'web3'
import { DICEADDRESS, DICEABI, CHAIN_ID} from '../config'

export default {
  name: "Header",

  data() {
    return {
      web3: new Web3(Web3.givenProvider),
      diceContract: null,
      walletAddress: "Connect Wallet",
      bscConnect: false,
      callInterval: null
    }
  },

  computed: {
    ...mapGetters({
      tokenBalance: 'tokenBalance'
    }),
    getWalletAdrAbbr() {
      if (this.walletAddress !== "Connect Wallet")
        return this.walletAddress.substr(0,9) + "......." + this.walletAddress.substr(-5, 8)
      else
        return "Connect Wallet"
    }
  },

  created() {
    this.diceContract = new this.web3.eth.Contract(DICEABI, DICEADDRESS)

    if (typeof window.ethereum === 'undefined') {
        alert('MetaMask is not installed!')
    }
    
    /***** when chain Network is changed *****/
    window.ethereum.on('chainChanged', (chainId) => {
      if (chainId !== CHAIN_ID) {
        alert('wrong network!')
        this.bscConnect = false
      } else {
        this.bscConnect = true;
      }
      this.changeWalletInfo()

      if (this.walletAddress !== "Connect Wallet") {
        this.getBalance()
      }
    });

    /***** when account is changed *****/
    window.ethereum.on('accountsChanged', (accounts) => {
      console.log('account changed')
      if (accounts[0]) {
        this.walletAddress = accounts[0]
        this.changeWalletInfo()
        this.getBalance()
      } else {
        this.walletAddress = "Connect Wallet"
        this.changeWalletInfo()
        this.getBalance()
      }
    })

    window.ethereum.request({
      method: 'eth_accounts'
    }).then((accounts) => {
      this.walletAddress = (accounts.length <= 0 ) ? "Connect Wallet" : accounts[0]
      if (accounts.length > 0) {
        if (window.ethereum.chainId === CHAIN_ID) {
          this.bscConnect = true
        }
          
        this.changeWalletInfo()
        this.getBalance()
        this.callInterval = setInterval(this.getBalance, 2500)
      }
    })
  },

  beforeDestroy() {
    clearInterval(this.callInterval);
  },

  methods: {
    ...mapMutations([
      'chageWalletState',
      'getBalance'
    ]),

    changeWalletInfo() {
      var walletState = {web3: this.web3, walletAddress: this.walletAddress, bscConnect: this.bscConnect, diceContract: this.diceContract}
      this.chageWalletState(walletState)
    },

    activatePage (type) {
      this.$emit('open-page', type)
    },

    async toggleWallet() {
      if (this.walletAddress !== "Connect Wallet") {
        var url = "https://etherscan.io/address/" + this.walletAddress
        window.open(url, '_blank')
        return
      }

      if (typeof window.ethereum === 'undefined') {
        alert('MetaMask is not installed!')
      }

      /*** check if it is on BSC network***/
      const chainId = await this.getChainId()
      if (chainId !== CHAIN_ID) {
        alert('wrong network!')
        this.bscConnect = false
        return
      }

      /*** metamask connecting ***/
      window.ethereum.request({ 
        method: 'eth_requestAccounts'
      }).then(() => {
        this.getBalance()
        this.callInterval = setInterval(this.getBalance, 2500)
      })
    },

    async getChainId() {
      const chainId = await window.ethereum.request({ method: 'eth_chainId' });
      return chainId
    }
  }
};

</script>

<style lang="scss">
.header {
  height: 60px;
  display: flex;
  box-sizing: border-box;
  justify-content: space-between;
  position: absolute;
  top: 0;
  width: 100%;
  z-index: 100;
  align-items: center;
  padding: 0 20px;

  .logo {
    display: flex;
    align-items: center;

    img {
      height: 36px;
    }
  }

  .actions {
    display: flex;
    align-items: center;

    button {
      margin: 0 5px;
      border: none;
      border-radius: 20px;
      border: 5px solid #120f1f;
      background: #2a2735;
      display: flex;
      align-items: center;
      font-size: 14px;
      padding: 3px 10px;
      color: #fff;
      text-overflow: none;

      img {
        margin-right: 10px;
      }

      span {
        line-height: -1;
      }
    }

    .wallet {
      margin: 0 5px;
      border: none;
      border-radius: 20px;
      background: #3064ad;
      display: flex;
      align-items: center;
      font-size: 14px;
      padding: 8px 10px;
      color: #fff;
    }
  }
}
</style>
