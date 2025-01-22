<template>
  <div
      class="min-h-screen flex items-center justify-center bg-gradient-to-br from-indigo-50 via-purple-100 to-pink-200 px-6">
    <div class="max-w-4xl w-full bg-white rounded-xl shadow-2xl p-8">
      <div class="text-center mb-8">
        <img src="https://img.icons8.com/ios-filled/50/000000/lock.png" alt="Vault Icon" class="mx-auto mb-4"/>
        <h1 class="text-3xl font-extrabold text-gray-800">🔒 Ansible Vault Encrypt/Decrypt</h1>
        <p class="text-gray-600 text-sm">Encrypt and decrypt your secrets with ease.</p>
      </div>

      <!-- Encryption Section -->
      <div class="mb-10">
        <h2 class="text-2xl font-semibold text-gray-700 mb-4">Encrypt Secret</h2>
        <div class="space-y-6">
          <div>
            <label for="vaultPassword" class="block text-sm font-medium text-gray-600">Vault Password</label>
            <input
                id="vaultPassword"
                type="password"
                v-model="vaultPassword"
                class="w-full mt-2 px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-purple-300 focus:outline-none"
                placeholder="Enter vault password"
            />
          </div>
          <div>
            <label for="secret" class="block text-sm font-medium text-gray-600">Secret</label>
            <textarea
                id="secret"
                v-model="secret"
                rows="4"
                class="w-full mt-2 px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-purple-300 focus:outline-none"
                placeholder="Enter secret to encrypt or crypted payload to decrypt"
                @paste="performCryptographicOperation($event.clipboardData.getData('text'))"
            ></textarea>
          </div>
          <button
              @click="performCryptographicOperation"
              class="w-full bg-purple-600 text-white py-3 rounded-lg hover:bg-purple-700 focus:ring-2 focus:ring-indigo-400 transition duration-150"
          >
            {{ action }}
          </button>
        </div>
        <div v-if="output" class="mt-6">
          <p class="text-sm font-medium text-gray-600">Encrypted Output:</p>
          <div
              class="bg-gray-100 text-sm p-4 rounded-lg overflow-x-auto border border-gray-300 relative"
          >
            <pre>{{ output }}</pre>
            <button
                :class="copyAnimation ? 'bg-green-500 text-white' : 'bg-purple-600 text-white'"
                @click="copyToClipboard(output)"
                class="absolute top-2 right-2 px-3 py-1 text-xs rounded-md transition duration-300 ease-in-out hover:bg-purple-700"
            >
              {{ copyAnimation ? 'Copied!' : 'Copy' }}
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import vault from "@/vault.js";
import {useToast} from "vue-toastification";

export default {
  name: "App",
  data() {
    return {
      vaultPassword: "",
      secret: "",
      output: "",
      action: "Encrypt/Decrypt",
      copyAnimation: false
    };
  },
  watch: {
    secret(newSecret) {
      if (newSecret.includes(vault.VAULT_HEADER)) {
        this.action = "Decrypt";
      } else if (newSecret.length > 0) {
        this.action = "Encrypt";
      } else {
        this.action = "Encrypt/Decrypt"
      }
    }
  },
  methods: {
    async performCryptographicOperation(text) {
      const toast = useToast();
      let tempSecret = this.secret;
      if (text.length) {
        tempSecret = text;
      }
      if (!this.vaultPassword || !tempSecret) {
        toast.error("Please provide both the vault password and the secret.");
        return;
      }
      if (!tempSecret.includes(vault.VAULT_HEADER)) {
        vault.encrypt(tempSecret, this.vaultPassword).then((encrypted) => {
          this.output = encrypted;
          toast.success("Secret encrypted successfully!");
        });
      } else {  // No Ansible header, must be something to encrypt
        vault.decrypt(tempSecret, this.vaultPassword).then((decryptedBuffer) => {
          this.output = decryptedBuffer;
          toast.success("Secret decrypted successfully!");
        })
            .catch((reason) => {
              toast.error(`An error occurred while decrypting the secret: ${reason}`);
            });
      }
    },
    copyToClipboard(content) {
      navigator.clipboard.writeText(content).then(() => {
        this.copyAnimation = true;
        setTimeout(() => {
          this.copyAnimation = false;
        }, 1500); // Reset the button after 1.5 seconds
      });
    },
  },
};
</script>

<style scoped>
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.animate-fade-in {
  animation: fadeIn 0.6s ease-out;
}

body {
  font-family: 'Inter', sans-serif;
  background-color: #f8f9fd;
}
</style>