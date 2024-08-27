<template>
  <div>
    <NuxtRouteAnnouncer />
    <NuxtWelcome />
    <div v-if="repoUrl">
      <button @click="handleOpenGitClient">Open Git Client</button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      repoUrl: null,
      detectedClients: []
    };
  },
  mounted() {
    const repoUrl = this.$route.query.repo;
    if (repoUrl && this.isValidGitHubUrl(repoUrl)) {
      this.repoUrl = repoUrl;
      this.detectGitClients().then(clients => {
        this.detectedClients = clients;
        if (clients.length === 0) {
          console.error('No supported Git clients found');
        }
      });
    } else {
      console.error('Invalid or missing repo URL');
    }
  },
  methods: {
    isValidGitHubUrl(url) {
      const regex = /^https:\/\/github\.com\/[\w-]+\/[\w-]+$/;
      return regex.test(url);
    },
    async detectGitClients() {
      const clients = [];
      // Check for GitHub Desktop
      if (await this.isClientAvailable('github-windows://')) {
        clients.push('GitHub Desktop');
      }
      // Check for GitKraken
      if (await this.isClientAvailable('gitkraken://')) {
        clients.push('GitKraken');
      }
      // Check for Git GUI (assuming a custom protocol or command)
      if (await this.isClientAvailable('git-gui://')) {
        clients.push('Git GUI');
      }
      return clients;
    },
    isClientAvailable(protocol) {
      return new Promise(resolve => {
        const iframe = document.createElement('iframe');
        iframe.style.display = 'none';
        iframe.src = protocol;
        document.body.appendChild(iframe);
        setTimeout(() => {
          document.body.removeChild(iframe);
          resolve(true); // Assume the client is available if no error occurs
        }, 1000);
      });
    },
    handleOpenGitClient() {
      if (this.detectedClients.length > 0) {
        this.openGitClient(this.repoUrl, this.detectedClients[0]); // Open with the first detected client
      } else {
        console.error('No supported Git clients found');
      }
    },
    openGitClient(url, client) {
      switch (client) {
        case 'GitHub Desktop':
          window.location.href = `github-windows://openRepo/${url}`;
          break;
        case 'GitKraken':
          window.location.href = `gitkraken://repo/${url}`;
          break;
        case 'Git GUI':
          window.location.href = `git-gui://openRepo/${url}`;
          break;
        default:
          console.error('Unsupported Git client');
      }
    }
  }
}
</script>

<style>
/* Add any custom styles here */
</style>