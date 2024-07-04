<template>
  <div id="app">
    <header>
      <h1>NFT Application</h1>
      <button @click="connectWallet">Connect Wallet</button>
    </header>
    <section v-if="walletConnected">
      <h2>My NFTs</h2>
      <div v-if="nfts.length">
        <div v-for="nft in nfts" :key="nft.tokenId">
          <img :src="nft.image" :alt="nft.name" />
          <p>{{ nft.name }}</p>
        </div>
      </div>
      <p v-else>No NFTs owned</p>
      <form @submit.prevent="mintNFT">
        <input v-model="metadata.name" placeholder="Name" required />
        <input v-model="metadata.image" placeholder="Image URL" required />
        <button type="submit">Mint NFT</button>
      </form>
    </section>
  </div>
</template>

<script>
import { ethers, Contract } from "ethers";
import contractAbi from "./contract-abi.json";

export default {
  data() {
    return {
      walletConnected: false,
      provider: null,
      signer: null,
      contract: null,
      account: null,
      contractWithSigner: null,
      nfts: [],
      metadata: {
        name: "",
        image: "",
      },
    };
  },
  methods: {
    async connectWallet() {
      if (window.ethereum) {
        try {
          await window.ethereum.request({ method: "eth_requestAccounts" });

          this.provider = new ethers.providers.Web3Provider();

          console.log(accounts);

          this.provider = await new ethers.providers.JsonRpcProvider(
            "https://data-seed-prebsc-1-s1.binance.org:8545/"
          );

          const accounts = await window.ethereum.request({
            method: "eth_requestAccounts",
          });

          this.signer = await this.provider.getSigner(accounts[0]);

          this.contract = new Contract(
            "0x31210A1453F4a02FC23C81BB6817d34b3d1B8D6d",
            contractAbi,
            this.provider,
            this.signer
          );

          this.provider = ethers.getDefaultProvider(
            "https://data-seed-prebsc-1-s1.binance.org:8545/"
          );

          this.signer = this.provider.getSigner();

          this.account = await this.signer.getAddress();

          this.contract = new ethers.Contract(
            "0x31210A1453F4a02FC23C81BB6817d34b3d1B8D6d",
            contractAbi,
            this.provider
          );

          this.walletConnected = true;
          this.loadNFTs();
        } catch (error) {
          console.error("Error connecting wallet:", error);
        }
      } else {
        alert("Please install MetaMask!");
      }
    },
    async loadNFTs() {
      const balance = await this.contract.balanceOf(this.account);
      const nfts = [];
      for (let i = 0; i < balance.toNumber(); i++) {
        const tokenId = await this.contract.tokenOfOwnerByIndex(
          this.account,
          i
        );
        const tokenUri = await this.contract.tokenURI(tokenId);
        const metadata = await fetch(tokenUri).then((res) => res.json());
        nfts.push({ tokenId, ...metadata });
      }
      this.nfts = nfts;
    },
    async mintNFT() {
      const response = await fetch("http://localhost:3000/upload-metadata", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(this.metadata),
      });

      const data = await response.json();

      const transaction = await this.contract.name();

      this.loadNFTs();
    },
  },
};
</script>

<style>
/* Add your CSS styles here */
</style>
