<script setup>
</script>

<template>
  <div>
    <div
      class="flex items-start sm:items-center justify-center min-h-screen font-sans subpixel-antialiased"
    >
      <div class="flex flex-col justify-center items-center">
        <div
          class="relative top-0 py-1 px-2 border-t-0 sm:border-t border-b-4 border-l border-r border-[#00cfff] bg-teal-200/30 rounded-b-lg rounded-t-none sm:rounded-t-lg"
        >
          <h1 class="text-lg font-thin">
            <strong>Search</strong> any public Repository
          </h1>
        </div>
        <div
          class="relative flex flex-col justify-center items-center space-y-2"
        >
          <form class="w-60" @submit.prevent="searchRepos">
            <div class="flex items-center border-b border-[#00cfff] py-2">
              <input
                class="appearance-none bg-transparent border-none w-40 text-gray-700 mr-3 py-1 px-2 leading-tight focus:outline-none"
                type="text"
                id="username"
                v-model="username"
                placeholder="Cobaya-chan"
                aria-label="Full name"
                required
              />
              <button
                class="bg-[#00cfff] w-full hover:bg-[#008bd6] border-[#00cfff] hover:border-[#008bd6] text-sm border-4 text-white py-1 px-2 transition-all duration-200 rounded"
                type="submit"
              >
                Find
              </button>
            </div>
          </form>

          <div v-if="repos" class="p-2">
            <div class="shadow-xl rounded-xl">
              <div class="flex flex-col border border-[#00cfff] rounded-xl">
                <div
                  class="p-8 md:p-4 flex flex-row justify-center flex-wrap md:flex-no-wrap min-w-full h-auto max-w-6xl"
                >
                  <div class="flex justify-center">
                    <ul class="divide-y divide-[#00cfff] w-full">
                      <li v-for="repo in repos" :key="repo.id">
                        <a :href="repo.html_url" target="_blank" rel="noopener noreferrer">{{ repo.name }}</a>
                        <i class="icon-link-ext text-sm text-[#00cfff]" />
                        <p class="text-sm font-normal tracking-tight">
                          {{ repo.description }}
                        </p>
                        <p class="text-sm font-normal">
                          <i class="icon-star text-sm text-[#00cfff]" />
                          {{ repo.stargazers_count }}
                        </p>
                        <p class="text-sm font-light tracking-widest">
                          <Text tid="repoUpdate" />
                          {{ new Date(repo.updated_at).toUTCString() }}
                        </p>
                        <p class="text-sm font-normal tracking-tight">
                          Commits: {{ repo.commitCount }}
                        </p>
                      </li>
                    </ul>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
        <div v-if="errorMsg">
          <p>{{ errorMsg }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      username: "",
      repos: null,
      errorMsg: "",
    };
  },
  methods: {
    async searchRepos() {
      this.repos = null;
      this.errorMsg = "";

      try {
        const response = await fetch(
          `https://api.github.com/users/${this.username}/repos`
        );

        if (response.ok) {
          const data = await response.json();
          const reposWithCommits = await Promise.all(
            data.map(async (repo) => {
              const commitsResponse = await fetch(repo.commits_url.replace("{/sha}", ""));
              if (commitsResponse.ok) {
                const commitsData = await commitsResponse.json();
                return { ...repo, commitCount: commitsData.length };
              } else {
                return { ...repo, commitCount: 0 };
              }
            })
          );
          this.repos = reposWithCommits;
        } else {
          throw new Error("Failed to fetch repositories");
        }
      } catch (error) {
        console.error(error);
        this.errorMsg = error.message;
      }
    },
  },
};
</script>