<script lang="ts">
  export let platforms: any[] = [];
  
  let query = '';
  
  // Fuzzy search reactivity
  $: filteredPlatforms = platforms.filter(p => {
    if (!query) return true;
    const lowerQuery = query.toLowerCase();
    return p.name.toLowerCase().includes(lowerQuery) || 
           p.category.toLowerCase().includes(lowerQuery) ||
           p.description.toLowerCase().includes(lowerQuery) ||
           p.tags.some((t: string) => t.toLowerCase().includes(lowerQuery));
  });

  // Strict sorting / taxonomy grouping requested by user (AI & Non-tech priority)
  $: groupedPlatforms = filteredPlatforms.reduce((acc, platform) => {
    const cat = platform.category;
    if (!acc[cat]) acc[cat] = [];
    acc[cat].push(platform);
    return acc;
  }, {} as Record<string, any[]>);

  // Hardcoded ordering for taxonomy UX hierarchy
  const categoryOrder = [
    "AI Training & Micro-Task",
    "Non-Tech / Support / VA",
    "Writing & Creative",
    "General Job Boards",
    "Tech & Development",
    "Resources & Meta-Directories"
  ];
</script>

<!-- The Search "Island" -->
<div class="mb-14 relative group max-w-2xl mx-auto">
  <div class="absolute inset-y-0 left-0 pl-4 flex items-center pointer-events-none">
    <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 text-ink/40 group-focus-within:text-accent transition-colors duration-200" fill="none" viewBox="0 0 24 24" stroke="currentColor">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
    </svg>
  </div>
  <input 
    type="text" 
    bind:value={query}
    placeholder="Fuzzy search 'Customer Support', 'Zero Experience', or 'Data Annotation'..."
    class="block w-full pl-12 pr-4 py-4 rounded-xl border border-ink/10 bg-white text-ink placeholder-ink/40 shadow-sm focus:outline-none focus:ring-2 focus:ring-accent/50 focus:border-accent transition-all duration-200 text-lg"
  />
  
  {#if query && filteredPlatforms.length > 0}
    <div class="absolute right-4 top-1/2 -translate-y-1/2 text-xs font-semibold text-accent/80 tracking-wider">
      {filteredPlatforms.length} MATCHES
    </div>
  {/if}
</div>

{#if filteredPlatforms.length === 0}
  <div class="text-center py-24">
    <h3 class="text-xl font-medium text-ink/70">No platforms found for "<span class="text-ink">{query}</span>"</h3>
    <p class="text-ink/50 mt-2">Try adjusting your filters or search terms.</p>
  </div>
{:else}
  <!-- The Categorized Grid -->
  <div class="space-y-20">
    {#each categoryOrder as category}
      {#if groupedPlatforms[category] && groupedPlatforms[category].length > 0}
        <section>
          <div class="mb-8 flex items-baseline justify-between border-b border-ink/10 pb-4">
            <h2 class="text-2xl font-bold tracking-tight text-ink flex items-center gap-3">
              <span class="w-2 h-6 bg-accent rounded-sm inline-block"></span>
              {category}
            </h2>
            <span class="text-sm font-medium text-ink/50 bg-ink/5 px-2 py-1 rounded-md">{groupedPlatforms[category].length} sites</span>
          </div>

          <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
            {#each groupedPlatforms[category] as platform (platform.id)}
              <a 
                href={platform.url} 
                target="_blank" 
                rel="noopener noreferrer" 
                class="group relative flex flex-col justify-between bg-white rounded-2xl p-6 shadow-sm border border-transparent hover:border-accent/40 hover:-translate-y-1 hover:shadow-md transition-all duration-300 h-full"
              >
                <!-- Platform Header -->
                <div>
                  <div class="flex items-start justify-between mb-4">
                    <h3 class="text-lg font-semibold text-ink leading-tight group-hover:text-accent transition-colors duration-200">
                      {platform.name}
                    </h3>
                    {#if platform.aiExposure === 'Low'}
                      <span class="inline-flex items-center px-2 py-0.5 rounded text-xs font-medium bg-green-50 text-green-700 border border-green-200/50">
                        AI-Safe
                      </span>
                    {:else if platform.aiExposure === 'Medium'}
                      <span class="inline-flex items-center px-2 py-0.5 rounded text-xs font-medium bg-yellow-50 text-yellow-700 border border-yellow-200/50">
                        Augmented
                      </span>
                    {/if}
                  </div>
                  <p class="text-sm text-ink/70 leading-relaxed line-clamp-3">
                    {platform.description}
                  </p>
                </div>

                <!-- Tags / Pills -->
                <div class="mt-6 flex flex-wrap gap-2">
                  {#each platform.tags as tag}
                    <span class="inline-flex items-center px-2 py-1 rounded-md text-[11px] font-semibold bg-parchment text-ink/60 uppercase tracking-widest border border-ink/5">
                      {tag}
                    </span>
                  {/each}
                </div>
              </a>
            {/each}
          </div>
        </section>
      {/if}
    {/each}
  </div>
{/if}
