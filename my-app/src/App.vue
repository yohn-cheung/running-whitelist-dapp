

<template>
  <main>
    <div>
      <h1 class="title">Welcome to Runners Club! #fitdev</h1>
      <div class="description">
        It is an NFT collection of runners in Crypto World
      </div>
      <div class="description">
        {{numberOfWhitelisted}} runner(s) have already joined the Whitelist
      </div>
      <div v-if="walletConnected">
        <div v-if="joinedWhitelist" class="description">
          Thanks for joining the Whitelist!
        </div>
        <div v-else-if="loading" class="description">
          Loading
        </div>
        <button v-else @click="addAddressToWhitelist" class="button">
          Join the Whitelist
        </button>
      </div>
      <div v-else>
        <button @click="connectWallet" class="button">
          Connect your wallet
        </button>
      </div>
    </div>
    <div>
      <img class="image" src="./assets/running-man.png" />
    </div>
  </main>
  <footer class="footer">
    Made by Yohn Cheung #fitdev
  </footer>
</template>

<script>
import Web3Modal from "web3modal";
import { providers, Contract } from "ethers";
import { WHITELIST_CONTRACT_ADDRESS, abi } from "./constants";


export default {
  name: "App",
  data() {
    return { 
      walletConnected: false, 
      joinedWhitelist: false,
      loading: false,
      numberOfWhitelisted: 0,
      web3ModalRef: null
    };
  },
  mounted() {
    if(!this.walletConnected) {
      this.web3ModalRef = new Web3Modal(
        {
          network: "rinkeby",
          providerOptions: {},
          disableInjectedProvider: false,
        }
      )
      this.connectWallet()
    }
  },
  methods: {
    async getProviderOrSigner(needSigner = false) {
      const provider = await this.web3ModalRef.connect();
      const web3Provider = new providers.Web3Provider(provider);

      const {chainId} = await web3Provider.getNetwork()

      if(chainId !==4) {
        window.alert("Change the network to Rinkeby")
        throw new Error("Chonge network to Rinkeby")
      }

      if (needSigner) {
        const signer = web3Provider.getSigner();
        return signer;
      }

      return web3Provider;
    },
    async addAddressToWhitelist() {
      try {
        const signer = await this.getProviderOrSigner(true)

        const whitelistContract = new Contract(
          WHITELIST_CONTRACT_ADDRESS,
          abi,
          signer
        );

        const tx = await whitelistContract.addAddressToWhitelist()
        this.loading = true;

        await tx.wait()
        this.loading = false
        await this.getNumberOfWhitelisted()
        this.joinedWhitelist = true
      } catch (error) {
        console.error(error)
      }
    },
    async getNumberOfWhitelisted() {
      try {
        const provider = await this.getProviderOrSigner()
        const whitelistContract = new Contract(
          WHITELIST_CONTRACT_ADDRESS, abi, provider
        )

        const _numberOfWhitelisted = await whitelistContract.numAddressesWhitelisted();
        this.numberOfWhitelisted = _numberOfWhitelisted;
      } catch (error) {
        console.error(error);
      }
    },
    async checkIfAddressInWhitelist() {
      try {
        const signer = await this.getProviderOrSigner(true)
        const whitelistContract = new Contract(WHITELIST_CONTRACT_ADDRESS, abi, signer);

        const address = await signer.getAddress()

        this.joinedWhitelist = await whitelistContract.whitelistedAddresses(address);
      } catch (error) {
        console.error(error);
      }
    },
    async connectWallet() {
      try {
        await this.getProviderOrSigner()
        this.walletConnected = true

        this.checkIfAddressInWhitelist()
        this.getNumberOfWhitelisted()
      } catch (error) {
        console.error(error)
      }
    },
  }
}

</script>

<style scoped>
main {
  min-height: 90vh;
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  
}

.footer {
  display: flex;
  padding: 2rem 0;
  border-top: 1px solid #eaeaea;
  justify-content: center;
  align-items: center;
}

.image {
  width: 70%;
  height: 50%;
  margin-left: 20%;
}

.title {
  font-size: 2rem;
  margin: 2rem 0;
}

.description {
  line-height: 1;
  margin: 2rem 0;
  font-size: 1.2rem;
}

.button {
  border-radius: 4px;
  background-color: blue;
  border: none;
  color: #ffffff;
  font-size: 15px;
  padding: 20px;
  width: 200px;
  cursor: pointer;
  margin-bottom: 2%;
}

@media (max-width: 1000px) {
  .main {
    width: 100%;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }
}
</style>
